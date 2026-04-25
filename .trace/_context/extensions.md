# Stage 2.4 Extension Points

## 概述

本專案的擴充設計以「開放/關閉」原則為核心：新增技能不需修改現有技能，只需新增一個目錄 + SKILL.md 即可。

## 主要擴充點

### 1. 新增 Skill（核心擴充點）

**位置**: `skills/<skill-name>/SKILL.md`

**規格**（`docs/skill-anatomy.md`、`CONTRIBUTING.md`）：

```bash
# 步驟
mkdir skills/my-new-skill
# 建立 SKILL.md，格式：
---
name: my-new-skill
description: Does X. Use when Y.  # ≤1024 chars
---
# My New Skill
## Overview
...（標準段落）
```

**規則**：
- `name` 必須與目錄名稱相同（kebab-case）
- Supporting files 只在超過 100 行時才建立（Progressive disclosure）
- Reference 材料放 `references/`，不放技能目錄內
- 不得複製其他技能的內容——以引用替代

**自動整合**：新增技能後，`using-agent-skills/SKILL.md` 的 flowchart 需要手動更新以涵蓋新技能（這是已知需要手動維護的點）。

### 2. 新增 Agent Persona

**位置**: `agents/<role>.md`

**規格**（`agents/README.md`）：

```yaml
---
name: role-name
description: Role description with trigger conditions.
---
```

**規則**：
- 一個 persona 只能有一個角色（single role, single output format）
- Personas **不呼叫其他 personas**（平台限制 + 架構原則）
- Persona 可以呼叫 skills（「技能是 how，persona 是 who」）
- 每個 persona 必須有 `Composition` block，說明何時直接呼叫 vs 透過 command 呼叫

**Claude Code 自動發現**：`agents/` 目錄中的 persona 在 plugin 安裝後自動可用（`subagent_type: code-reviewer` 等）。

**不支援的 frontmatter**（plugin 環境下靜默忽略）：`hooks`、`mcpServers`、`permissionMode`。

### 3. 新增 Slash Command

**位置**: `.claude/commands/<name>.md`

**格式**：
```yaml
---
description: Brief description shown in command list
---
# Command instructions...
```

**規則**：
- Command 是 orchestration layer（「the when」）
- Command 組合 personas 和 skills，但本身不實作 domain logic
- 唯一允許的多 persona 編排模式：parallel fan-out（如 `/ship`）
- 不要建立「路由 persona」（決定呼叫哪個 persona 的 meta-agent）—— 這是反模式

### 4. 新增 Reference Checklist

**位置**: `references/<name>.md`

**規格**：
- 技能使用時按需載入的補充材料
- 不放在技能目錄內（避免自動載入造成 token 浪費）
- 適合長度超過 50 行的 patterns、checklist、詳細規範

目前已有：
- `testing-patterns.md`（TDD 補充）
- `security-checklist.md`（security-and-hardening 補充）
- `performance-checklist.md`（performance-optimization 補充）
- `accessibility-checklist.md`（frontend-ui-engineering 補充）
- `orchestration-patterns.md`（agents/ 設計補充）

### 5. 新增 Hook

**設定位置**: `hooks/hooks.json`（全局）或 `.claude/settings.json`（專案層級）

**Claude Code 支援的 Hook 類型**：
- `SessionStart`：session 開始時執行
- `PreToolUse`：工具調用前執行（可攔截）
- `PostToolUse`：工具調用後執行（異步）

**現有 hooks**：
- `SessionStart` → `hooks/session-start.sh`（自動注入 meta-skill）
- `PreToolUse WebFetch` → `hooks/sdd-cache-pre.sh`（SDD cache，選用）
- `PostToolUse WebFetch` → `hooks/sdd-cache-post.sh`（SDD cache，選用）
- `PreToolUse` / `PostToolUse` 用於 simplify-ignore（`hooks/simplify-ignore.sh`）

**Hook 退出碼語意**：
- `exit 0`：允許工具繼續執行
- `exit 2`：攔截工具調用，stderr 內容作為工具結果回傳給 agent

### 6. 新增平台支援

**機制**：新增 `docs/<platform>-setup.md` + 在 README.md 新增 `<details>` 折疊區塊。

目前支援的平台：
- Claude Code（原生 Plugin + slash commands）
- Cursor（`.cursor/rules/` 或 `.cursorrules`）
- Gemini CLI（`gemini skills install`）
- Windsurf（rules 設定）
- GitHub Copilot（`.github/copilot-instructions.md`）
- OpenCode（AGENTS.md skill 機制）
- Kiro IDE（`.kiro/skills/`）

### 7. OpenCode 格式擴充（不同格式規格）

`AGENTS.md` 描述的 OpenCode 格式與 Claude Code 格式不同，支援額外的擴充點：

```
skills/<skill-name>/         # 目錄
  SKILL.md                   # 技能定義（同格式）
  scripts/<script>.sh        # 可執行腳本（OpenCode 額外要求）
skills/<skill-name>.zip      # 打包發布（OpenCode 額外要求）
```

## 限制與約束

| 約束 | 來源 | 說明 |
|------|------|------|
| SKILL.md 建議 ≤ 500 行 | `AGENTS.md` | 超過時 split supporting files |
| description ≤ 1024 字元 | `docs/skill-anatomy.md` | 注入 system prompt |
| 禁止複製內容 | `CONTRIBUTING.md` | 用引用替代 |
| Sub-agents 不能呼叫 sub-agents | Claude Code 平台限制 | 強制單層編排 |
| Personas 不呼叫 Personas | `agents/README.md` 架構原則 | 防止路由 meta-agent 反模式 |
| Hooks 不支援 mcpServers/permissionMode | Claude Code plugin 限制 | 靜默忽略 |

## 「如何新增功能而不動核心」

1. **新增技能**：在 `skills/` 建立目錄 + SKILL.md，不需修改任何現有檔案
2. **新增 persona**：在 `agents/` 建立 Markdown，自動可用
3. **新增 command**：在 `.claude/commands/` 建立 Markdown，自動掛載
4. **手動更新** `using-agent-skills/SKILL.md` 的 flowchart（這是目前唯一需要手動同步的核心檔案）
