# agent-skills 專案總覽

## 一句話摘要

**agent-skills** 是一套由 Addy Osmani 開發的生產等級 Markdown 工程工作流程集合，讓 AI coding agents（Claude Code、Cursor、Gemini CLI 等）能夠遵循資深工程師的開發紀律——涵蓋從想法精煉到部署上線的完整軟體開發生命週期。

## 技術棧總覽

| 類別 | 技術 | 版本 | 用途 |
|------|------|------|------|
| 主要格式 | Markdown + YAML | — | Skill 定義（SKILL.md 含 frontmatter） |
| 腳本語言 | Bash | 3.2+ | Session hooks、SDD cache、idea-refine 初始化 |
| Plugin 格式 | JSON | — | Claude Code plugin manifest |
| CI/CD | GitHub Actions | — | Plugin 結構驗證與安裝測試 |
| CLI 工具 | @anthropic-ai/claude-code | latest | CI 驗證環境 |
| 外部工具依賴 | jq, curl, shasum | — | SDD cache hook 執行環境 |
| 版本控制 | Git | — | 主分支 `main`，開發分支 `claude/trace-codebase-docs-CKc42` |

## 關鍵指令速查

| 動作 | 指令 / 方式 |
|------|------------|
| Claude Code 安裝 | `/plugin marketplace add addyosmani/agent-skills` → `/plugin install agent-skills@addy-agent-skills` |
| 本地開發安裝 | `git clone https://github.com/addyosmani/agent-skills.git` → `claude --plugin-dir /path/to/agent-skills` |
| Cursor 安裝 | `cp skills/*/SKILL.md .cursor/rules/` |
| Gemini CLI 安裝 | `gemini skills install https://github.com/addyosmani/agent-skills.git --path skills` |
| Gemini CLI slash commands | 從專案根目錄執行 `gemini`，自動發現 `.gemini/commands/` 下的 7 個指令（使用 `/planning` 而非 `/plan`） |
| 驗證 plugin 結構 | `claude plugin validate .` |
| 執行 CI 測試 | Push to GitHub → GitHub Actions 自動觸發 |
| 啟用 SDD cache | 手動在 `.claude/settings.json` 新增 PreToolUse/PostToolUse hook |
| 新增技能 | `mkdir skills/<name>` → 建立 `SKILL.md` → 更新 `using-agent-skills/SKILL.md` flowchart |
| 打包技能（OpenCode）| `cd skills && zip -r <skill-name>.zip <skill-name>/` |

## Slash Commands 速查

| 指令 | 用途 | 對應技能 |
|------|------|---------|
| `/spec` | 撰寫結構化規格，避免倉促寫程式碼 | spec-driven-development |
| `/plan` | 分解任務為可驗證單元 | planning-and-task-breakdown |
| `/build` | 逐步實作並測試 | incremental-implementation + TDD |
| `/test` | TDD 工作流程 / Prove-It pattern | test-driven-development |
| `/review` | 五軸程式碼審查 | code-review-and-quality |
| `/code-simplify` | 簡化程式碼，保持行為不變 | code-simplification |
| `/ship` | 並行 fan-out 審查，產出 go/no-go 決策 | shipping-and-launch + 3 personas |

## 文件地圖

| 文件 | 路徑 | 說明 |
|------|------|------|
| 本頁（總覽） | `.trace/INDEX.md` | 速查入口 |
| 程式碼地圖 | `.trace/CODEBASE_MAP.md` | 目錄說明 + 「我想改 X 看哪裡？」 |
| 系統架構 | `.trace/ARCHITECTURE.md` | 架構圖、組件關係、設計決策 |
| 資料模型 | `.trace/DATA_MODEL.md` | Skill/Persona/Command 結構 |
| API 介面 | `.trace/API_SURFACE.md` | 所有公開介面與設定規格 |
| 開發指南 | `.trace/DEV_GUIDE.md` | 環境建置、貢獻流程 |
| 探索紀錄 | `.trace/DISCOVERY_LOG.md` | 待解問題、技術債、落差分析 |

## 專案術語表

| 術語 | 定義 |
|------|------|
| **Skill** | 一個工程工作流程，以 `SKILL.md` 表達，包含步驟、驗證標準和防口號化表格 |
| **SKILL.md** | Skill 的定義檔案，含 YAML frontmatter（name + description）和標準段落結構 |
| **Persona** | 特定角色的 agent，以 `agents/<role>.md` 表達（例：`code-reviewer`）|
| **Slash Command** | 用戶觸發的工作流入口，以 `.claude/commands/<name>.md` 表達 |
| **SessionStart hook** | Plugin 安裝後每次 session 開始時自動執行的腳本，注入 meta-skill |
| **Progressive disclosure** | 技能按需載入，不全部預載，降低 token 消耗 |
| **Anti-rationalization** | Skill 中的「Common Rationalizations」表格，防止 agent 自我合理化跳過步驟 |
| **Fan-out** | `/ship` 使用的並行多 persona 編排模式 |
| **SDD cache** | `source-driven-development` 技能的 WebFetch 快取，基於 HTTP ETag 驗證 |
| **Meta-skill** | `using-agent-skills`，技能發現導覽，每個 session 自動注入 |
| **Five-axis review** | `code-review-and-quality` 的五維審查：Correctness / Readability / Architecture / Security / Performance |
| **Prove-It pattern** | `test-driven-development` 用於 bug fix 的模式：先寫失敗測試，確認失敗後再修復 |
| **Gated workflow** | `spec-driven-development` 的四階段閘控流程，每階段須人類確認才能推進 |
| **Vertical slice** | `incremental-implementation` 的切片策略：每次實作一條完整的 stack 路徑 |
| **Trunk-based development** | `git-workflow-and-versioning` 採用的 Git 工作流，主幹直接提交，避免長命分支 |
| **Shift Left** | `ci-cd-and-automation` 原則，品質檢查越早越好 |
| **Chesterton's Fence** | `code-simplification` 原則，不理解原因前不輕易刪除程式碼 |
| **Hyrum's Law** | `api-and-interface-design` 引用，所有可觀察的 API 行為都會被某人依賴 |
| **Beyonce Rule** | `test-driven-development` 引用，你放進去的功能，你得測試它 |
