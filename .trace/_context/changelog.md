# Changelog：1f66d57..19e49a0

更新日期：2026-04-30

## Commit 歷史摘要

| Commit | 說明 |
|--------|------|
| `36d26a6` | feat: add symlink to make opencode skills work properly |
| `54cc926` | feat: add Gemini CLI slash command support |
| `43a0dde` | fix: JSON-escape hook content in session-start hook |
| `501d226` | hooks: gracefully fall back when jq is missing |
| `e6d4005` | Align Gemini CLI commands with official schema and subagent model |

## 變更統計

- **變更檔案數**：11（8 新增 + 3 修改）
- **插入行數**：+210
- **刪除行數**：-8

## 逐檔說明

### 新增：`.gemini/commands/` 目錄（7 個 TOML 檔案）

Gemini CLI 原生 slash commands，TOML 格式（與 Claude Code 的 YAML+Markdown 不同）：

| 檔案 | 對應功能 | 備注 |
|------|---------|------|
| `spec.toml` | /spec | 與 Claude Code 版本邏輯一致 |
| `planning.toml` | /planning | **注意：命名為 `/planning` 而非 `/plan`**（/plan 與 Gemini CLI 內建指令衝突） |
| `build.toml` | /build | 同 Claude Code |
| `test.toml` | /test | 同 Claude Code |
| `review.toml` | /review | 同 Claude Code |
| `code-simplify.toml` | /code-simplify | 同 Claude Code |
| `ship.toml` | /ship | Fan-out 邏輯適配 Gemini CLI 的 subagent 模型（`.gemini/agents/` 目錄） |

**TOML 格式規格**：
```toml
description = "一句話描述"
prompt = """
執行指令的完整說明...
"""
```

**`/ship` 的 Gemini CLI 差異**：
- Sub-agent 呼叫方式：Gemini CLI 中 `.gemini/agents/` 下的 persona 以 tool 名稱呼叫
- `@code-reviewer` 語法可顯式觸發
- 若 sub-agents 不可用，退化為循序執行（仍可運作）

### 新增：`.opencode/skills` symlink

`/home/user/agent-skills/.opencode/skills -> ../skills/`

讓 OpenCode 透過符號連結自動發現 `skills/` 目錄下的所有技能，不再需要手動設定路徑。

### 修改：`hooks/session-start.sh`（2 個 bug fix）

**Fix 1 - JSON injection 修復**（`43a0dde`）：
- 舊：heredoc 字串串接（`$CONTENT` 若含 `"` 或 `\` 會破壞 JSON 結構）
- 新：使用 `jq -cn --arg message "..." '...'` 正確 escape JSON

**Fix 2 - jq 缺失時 graceful fallback**（`501d226`）：
- 新增 `command -v jq` 檢查
- 若 jq 不存在，輸出 INFO 訊息而非錯誤，允許 session 繼續
- 訊息提示用戶安裝 jq（`brew install jq` / `apt-get install jq`）

### 修改：`docs/gemini-cli-setup.md`

新增「Slash Commands」章節，說明 `.gemini/commands/` 下 7 個指令的對應功能，以及 `/planning` vs `/plan` 的命名注意事項。

### 修改：`README.md`

專案結構圖的 comments 更新：
- `.claude/commands/` 改標注為 `7 slash commands (Claude Code)`
- 新增 `.gemini/commands/` 條目：`7 slash commands (Gemini CLI)`

## 變更分類

| 變更 | 類別 | 影響的 .trace 文件 |
|------|------|-----------------|
| `.gemini/commands/` 7 個新檔案 | 新平台整合 + Extension Points | CODEBASE_MAP, ARCHITECTURE, API_SURFACE |
| `.opencode/skills` symlink | 整合方式改變 | CODEBASE_MAP, ARCHITECTURE, DEV_GUIDE |
| `hooks/session-start.sh` bug fix | Bug fix（hook 機制） | ARCHITECTURE, DEV_GUIDE |
| `docs/gemini-cli-setup.md` 更新 | 文件更新 | DEV_GUIDE |
| `README.md` 微調 | 無影響 | 無 |

## 變更幅度評估

- 總檔案數（不含 .trace/）：~73
- 有意義的變更檔案：10（排除 README 微調）
- **變更幅度：≈ 14%（中度更新）**
- **判定：繼續增量更新**
