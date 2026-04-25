# Stage 2.5 外部整合

## 概述

本專案本身不呼叫外部 API（它是文件集合），但作為 plugin 需要整合到多個外部平台和工具鏈。此外，SDD cache hook 涉及 HTTP 整合邏輯。

## 平台整合清單

### 1. Claude Code（主要整合）

**整合方式**: Plugin + SessionStart Hook + Slash Commands

**安裝路徑**：
```bash
/plugin marketplace add addyosmani/agent-skills
/plugin install agent-skills@addy-agent-skills
```

**Protocol**：Claude Code Plugin 協定
- 讀取 `.claude-plugin/plugin.json`（名稱、版本、commands 路徑）
- 讀取 `.claude-plugin/marketplace.json`（marketplace 識別碼）
- 自動掛載 `.claude/commands/` 目錄
- 執行 `hooks/hooks.json` 中定義的 SessionStart hook

**驗證 CI**（`.github/workflows/test-plugin-install.yml`）：
```yaml
- run: claude plugin validate .         # 驗證 manifest 結構
- run: claude plugin marketplace add ./ # 本地 marketplace 測試
- run: claude plugin install agent-skills@addy-agent-skills --scope user
```

**失敗處理**：SessionStart hook 若找不到 meta-skill，輸出 INFO 級別提示而非錯誤（graceful degradation）。

### 2. Cursor

**整合方式**: Rules 目錄 / `.cursorrules` 文件 / Notepads

**設定**（`docs/cursor-setup.md`）：
```bash
mkdir -p .cursor/rules
cp agent-skills/skills/*/SKILL.md .cursor/rules/
```

**無失敗處理**：Cursor 靜態載入 rules，無動態錯誤恢復機制。

### 3. Gemini CLI

**整合方式**: `gemini skills install` 指令

```bash
# 從 GitHub 安裝
gemini skills install https://github.com/addyosmani/agent-skills.git --path skills
# 從本地安裝
gemini skills install ./agent-skills/skills/
```

**Protocol**：Gemini CLI skills 協定（自動發現 `skills/` 目錄下的 SKILL.md 檔案）。

### 4. Windsurf

**整合方式**: Windsurf rules 設定（詳見 `docs/windsurf-setup.md`）。

### 5. GitHub Copilot

**整合方式**: `.github/copilot-instructions.md` + `agents/` 作為 Copilot personas

**設定**（`docs/copilot-setup.md`）：
- Agent personas → Copilot persona 定義
- SKILL.md 內容 → `.github/copilot-instructions.md` 中的指令

### 6. OpenCode

**整合方式**: AGENTS.md 意圖映射 + `skill` tool

**協定**：OpenCode skill tool 協定（`AGENTS.md` 完整說明）：
```
用戶意圖 → OpenCode 讀取 AGENTS.md → 映射到 skills/<name>/SKILL.md → 執行
```

### 7. Kiro IDE

**整合方式**: `.kiro/skills/` 目錄（Project 或 Global 層級）。

## SDD Cache HTTP 整合

`source-driven-development` 技能透過 hook 整合 HTTP 快取協定：

**依賴工具**：
- `curl`（HTTP 請求）
- `jq`（JSON 解析）
- `shasum` 或 `sha256sum`（快取鍵計算）

**整合邏輯**（`hooks/sdd-cache-pre.sh`）：

```bash
# 快取驗證：對目標 URL 發送 HEAD 請求
curl --silent --max-time 5 \
  -H "If-None-Match: $ETAG" \
  -H "If-Modified-Since: $LAST_MOD" \
  "$URL" -o /dev/null -w "%{http_code}"
```

**外部依賴行為**：

| 情境 | 處理方式 |
|------|---------|
| curl 不存在 | `exit 0`（graceful degradation，允許 WebFetch 繼續） |
| jq 不存在 | `exit 0`（graceful degradation） |
| shasum/sha256sum 不存在 | `exit 0`（graceful degradation） |
| 伺服器回 304 Not Modified | `exit 2`（攔截 WebFetch，返回快取） |
| 伺服器回 200 OK | `exit 0`（允許 WebFetch 重新執行） |
| 伺服器無 ETag/Last-Modified | 永不快取該 URL |
| 網路超時 | ⚠️ 未驗證完整行為，`curl --max-time 5` 應有 5 秒超時 |

**快取格式**（`.claude/sdd-cache/<hash>.json`）：
```json
{
  "url": "https://...",
  "prompt": "original prompt used",
  "etag": "W/\"abc123\"",
  "last_modified": "Tue, 01 Jan 2026 00:00:00 GMT",
  "content": "cached WebFetch response",
  "fetched_at": "2026-01-01T00:00:00Z"
}
```

**已知限制**（`hooks/SDD-CACHE.md`）：
- 每次快取寫入需要額外一次 HEAD 請求（Claude Code 未暴露已收到的 response headers）
- 無團隊共享快取
- Prompt 相同 URL 不同 prompt 共用同一快取條目（prompt-shaped body）

## GitHub Actions CI 整合

**工作流程**（`.github/workflows/test-plugin-install.yml`）：

```yaml
on: [push, pull_request, workflow_dispatch]

jobs:
  validate:
    - 安裝 @anthropic-ai/claude-code
    - 執行 `claude plugin validate .`（驗證 manifest）
  
  test-install:
    needs: validate
    - 安裝 @anthropic-ai/claude-code
    - 設定 HTTPS clone（`git config --global url.https://...`）
    - 執行 marketplace add + install 完整流程
```

**外部依賴**：
- `npm install -g @anthropic-ai/claude-code`
- GitHub Actions `ubuntu-latest` runner
- `actions/checkout@v6`

## 平台整合對照表

| 平台 | 安裝複雜度 | Skills 自動發現 | Slash Commands | Personas | Hooks |
|------|-----------|----------------|----------------|---------|-------|
| Claude Code | 簡單（2 指令） | ✅ SessionStart | ✅ 自動 | ✅ subagents | ✅ |
| Cursor | 手動複製 | ❌ 需手動 | ❌ | ❌ | ❌ |
| Gemini CLI | 1 指令 | ✅ | ❌ | ❌ | ❌ |
| Windsurf | 手動設定 | ❌ | ❌ | ❌ | ❌ |
| Copilot | 手動設定 | ❌ | ❌ | 部分 | ❌ |
| OpenCode | AGENTS.md | ✅ skill tool | ❌ | 部分 | ❌ |
| Kiro IDE | 複製目錄 | ⚠️ 未驗證 | ❌ | ❌ | ❌ |
