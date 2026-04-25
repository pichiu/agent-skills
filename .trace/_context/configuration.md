# Stage 2.6 設定與環境

## 設定載入優先順序

本專案沒有傳統的環境變數設定機制，但 Claude Code plugin 有分層設定系統：

```
用戶層級 (~/.claude/settings.json)         ← 最高優先
    ↓ 覆蓋
專案層級 (.claude/settings.json)
    ↓ 覆蓋
Plugin 層級 (hooks/hooks.json)              ← 最低優先
```

## Plugin 設定

### `.claude-plugin/plugin.json`

```json
{
  "name": "agent-skills",
  "description": "Production-grade engineering skills...",
  "version": "1.0.0",
  "author": { "name": "Addy Osmani" },
  "homepage": "https://github.com/addyosmani/agent-skills",
  "repository": "https://github.com/addyosmani/agent-skills",
  "license": "MIT",
  "commands": "./.claude/commands"
}
```

**關鍵欄位**：
- `commands`：slash commands 目錄路徑
- `version`：目前 `1.0.0`

### `.claude-plugin/marketplace.json`

```json
{
  "name": "addy-agent-skills",     ← marketplace 安裝識別碼
  "owner": { "name": "Addy Osmani" },
  "plugins": [{
    "name": "agent-skills",
    "source": { "source": "github", "repo": "addyosmani/agent-skills" }
  }]
}
```

**注意**：`plugin.json` 的 `name` 是 `"agent-skills"`，但 marketplace 安裝識別碼是 `"addy-agent-skills"`（帶前綴）。安裝指令：`/plugin install agent-skills@addy-agent-skills`。

## Hooks 設定

### `hooks/hooks.json`（Plugin 全局 Hook）

```json
{
  "hooks": {
    "SessionStart": [{
      "hooks": [{
        "type": "command",
        "command": "bash ${CLAUDE_PLUGIN_ROOT}/hooks/session-start.sh"
      }]
    }]
  }
}
```

**環境變數**：
- `$CLAUDE_PLUGIN_ROOT`：plugin 安裝路徑（Claude Code 注入）

### SDD Cache Hook（選用，`.claude/settings.json` 配置）

此 hook **預設未啟用**，需要用戶手動在 `.claude/settings.json` 或 `.claude/settings.local.json` 中設定：

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "WebFetch",
      "hooks": [{
        "type": "command",
        "command": "bash ${CLAUDE_PROJECT_DIR}/hooks/sdd-cache-pre.sh",
        "timeout": 10
      }]
    }],
    "PostToolUse": [{
      "matcher": "WebFetch",
      "hooks": [{
        "type": "command",
        "command": "bash ${CLAUDE_PROJECT_DIR}/hooks/sdd-cache-post.sh",
        "async": true,
        "timeout": 10
      }]
    }]
  }
}
```

**環境變數**：
- `$CLAUDE_PROJECT_DIR`：專案根目錄（Claude Code 注入）
- `SDD_CACHE_DEBUG`：設為 `1` 啟用 debug logging
- Debug sentinel file：`.claude/sdd-cache/.debug`（touch 啟用，rm 停用）

### Simplify-Ignore Hook（選用）

**相關檔案**：
- `hooks/SIMPLIFY-IGNORE.md`：說明文件
- `hooks/simplify-ignore.sh`：設定豁免清單
- `hooks/simplify-ignore-test.sh`：測試腳本

設定方式同 SDD cache，在 `.claude/settings.json` 中手動啟用。

## Feature Flags

本專案無傳統 feature flags，但有幾個「opt-in」機制：

| 功能 | 啟用方式 | 說明 |
|------|---------|------|
| SDD Cache | `.claude/settings.json` 手動加 hook | WebFetch 快取，避免重複抓文件 |
| Simplify-Ignore | `.claude/settings.json` 手動加 hook | 指定哪些程式碼不進行簡化 |
| Debug logging（SDD）| `SDD_CACHE_DEBUG=1` 或 touch `.debug` | SDD cache 的詳細 log |

## 路徑設定依賴

| 設定 | 路徑變數 | 覆蓋方式 |
|------|---------|---------|
| Hook 腳本路徑 | `${CLAUDE_PLUGIN_ROOT}` | 全局安裝時自動解析 |
| SDD cache 路徑 | `${CLAUDE_PROJECT_DIR}/.claude/sdd-cache/` | `CLAUDE_PROJECT_DIR` 環境變數 |
| Hooks 路徑（本地開發） | 需改為絕對路徑 | 說明在 `SDD-CACHE.md` |

**本地開發注意**（`hooks/SDD-CACHE.md`）：
> 若安裝在共享位置（如 `~/agent-skills`），將 `${CLAUDE_PROJECT_DIR}/hooks/...` 替換為絕對路徑。

## Secrets 管理

本專案不處理任何 secrets。但 SDD cache hook 的 `sdd-cache-pre.sh` 會讀取 WebFetch 工具的輸入（包含 URL），需確保 URL 不含敏感憑證。

## `.gitignore` 設定

```gitignore
.DS_Store
node_modules/
.env
.env.*
*.log
.claude/.simplify-ignore-cache/   ← simplify-ignore cache
.claude/sdd-cache/                 ← SDD cache（快取不提交）
```

## CI/CD 環境設定

**需求**（`.github/workflows/test-plugin-install.yml`）：
- Node.js + npm（用於安裝 `@anthropic-ai/claude-code`）
- git（需要配置 HTTPS clone，避免 SSH 問題）

```yaml
- name: Configure git to use HTTPS
  run: git config --global url."https://github.com/".insteadOf "git@github.com:"
```

**無 secrets 需求**：CI 工作流不需要任何 GitHub token 或 API key（plugin 驗證和安裝走公開路徑）。
