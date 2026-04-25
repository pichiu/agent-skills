# Stage 2.1 Entry Points

本專案非傳統可執行程式，「entry points」指的是 AI agent 如何「啟動」並載入 skills 的入口機制。

## 主要入口機制

### 1. SessionStart Hook（自動注入）

**檔案**: `hooks/hooks.json` + `hooks/session-start.sh`

```
Claude Code 啟動 session
    ↓
讀取 hooks/hooks.json
    ↓
執行 SessionStart hook: bash hooks/session-start.sh
    ↓
讀取 skills/using-agent-skills/SKILL.md 內容
    ↓
包裝成 JSON: {"priority": "IMPORTANT", "message": "agent-skills loaded. ..."}
    ↓
注入到 agent 的 context（優先順序: IMPORTANT）
```

`session-start.sh` 關鍵邏輯（`hooks/session-start.sh`）：
```bash
CONTENT=$(cat "$META_SKILL")
cat <<EOF
{
  "priority": "IMPORTANT",
  "message": "agent-skills loaded. Use the skill discovery flowchart...\n\n$CONTENT"
}
EOF
```

這個 hook 確保每個新 session 都自動獲得 meta-skill（技能發現導覽），讓 agent 知道「有哪些 skills 可用、何時使用」。

### 2. Plugin Install（Claude Code Marketplace）

**檔案**: `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`

安裝路徑：
```
用戶執行:
  /plugin marketplace add addyosmani/agent-skills
  /plugin install agent-skills@addy-agent-skills
    ↓
Claude Code 讀取 .claude-plugin/plugin.json
  → name: "agent-skills"
  → commands: "./.claude/commands"
    ↓
Plugin 安裝完成，slash commands 可用
    ↓
SessionStart hook 自動綁定（透過 hooks.json）
```

`plugin.json` 結構（`.claude-plugin/plugin.json`）：
```json
{
  "name": "agent-skills",
  "version": "1.0.0",
  "author": { "name": "Addy Osmani" },
  "repository": "https://github.com/addyosmani/agent-skills",
  "commands": "./.claude/commands"
}
```

### 3. Slash Commands（用戶主動觸發）

**目錄**: `.claude/commands/`

7 個 slash command，每個都是一個 YAML frontmatter + 內容的 Markdown 檔案：

| Command | 檔案 | 對應技能 |
|---------|------|---------|
| `/spec` | `spec.md` | spec-driven-development |
| `/plan` | `plan.md` | planning-and-task-breakdown |
| `/build` | `build.md` | incremental-implementation + TDD |
| `/test` | `test.md` | test-driven-development |
| `/review` | `review.md` | code-review-and-quality |
| `/code-simplify` | `code-simplify.md` | code-simplification |
| `/ship` | `ship.md` | shipping-and-launch（fan-out 編排） |

每個 command 的 frontmatter 格式（例：`spec.md`）：
```yaml
---
description: Start spec-driven development — write a structured specification before writing code
---
```

`/ship` 是最複雜的 command，它實作 **parallel fan-out 編排模式**：
1. 同時派發 3 個 sub-agent（`code-reviewer`、`security-auditor`、`test-engineer`）
2. 等待所有報告
3. 在 main context 中合併為 go/no-go 決策

### 4. AGENTS.md 意圖映射（OpenCode）

**檔案**: `AGENTS.md`

OpenCode 不支援 slash commands，改用意圖映射：
```
用戶意圖 → Agent 分析 AGENTS.md → 找到對應技能 → 執行工作流
```

例：用戶說「我要新增一個功能」→ Agent 識別為 `spec-driven-development` + `incremental-implementation` + `test-driven-development`。

### 5. 手動載入（Universal 方式）

**說明**: `docs/getting-started.md`

最通用的入口，適用所有 agent：
- 將 SKILL.md 內容貼入 system prompt
- 將 SKILL.md 複製到 `.cursor/rules/` 等 rules 目錄
- 在對話中直接引用技能名稱

## Initialization 流程

```
Plugin 安裝（一次性）
    ↓
Session 開始
    ↓
SessionStart hook 執行（session-start.sh）
    ↓  
注入 using-agent-skills meta-skill
    ↓
用戶發出指令 or 觸發 slash command
    ↓
Agent 透過 meta-skill 的 flowchart 識別適用技能
    ↓
讀取對應 SKILL.md（progressive disclosure）
    ↓
執行技能工作流程
```

## 技能的自動觸發

根據 README 和 AGENTS.md，skills 也可以「根據用戶行為自動觸發」：
- 設計 API → 觸發 `api-and-interface-design`
- 建立 UI → 觸發 `frontend-ui-engineering`
- 修 bug → 觸發 `debugging-and-error-recovery`

這種自動觸發由 agent 的判斷邏輯驅動，而非程式碼判斷，meta-skill 中的 flowchart 是關鍵。

## SDD Cache Hook（選用功能）

**檔案**: `hooks/sdd-cache-pre.sh`, `hooks/sdd-cache-post.sh`

額外的 PreToolUse / PostToolUse hook，專門為 `source-driven-development` 技能加速：
- 在 WebFetch 執行前檢查 URL 是否有快取（HTTP ETag/Last-Modified 驗證）
- 若 304 Not Modified，直接返回快取內容（bypass WebFetch）
- PostToolUse 時儲存新的快取條目

快取路徑：`.claude/sdd-cache/<sha256(url)>.json`（已加入 `.gitignore`）
