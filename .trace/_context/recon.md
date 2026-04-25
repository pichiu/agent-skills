# Stage 1 偵察報告

## 專案識別

- **名稱**: agent-skills
- **維護者**: Addy Osmani (Google Chrome 工程師)
- **授權**: MIT
- **儲存庫**: https://github.com/addyosmani/agent-skills
- **版本**: 1.0.0
- **GitHub 星數**: ~22,300（截至 2026-04）
- **Forks**: ~2,800

## 一句話摘要

一套生產等級的 Markdown 格式工程工作流程集合（稱為「skills」），讓 AI coding agent（Claude Code、Cursor、Gemini CLI 等）能夠遵循資深工程師的開發紀律——從 spec 撰寫到部署上線的完整生命週期。

## 架構模式

**Plugin-based / Documentation-only project**

- 無可執行的伺服器或應用程式邏輯
- 核心產物全是 Markdown 文件 (`.md`)
- 少量 Bash 腳本用於 hook 機制
- 透過 Claude Code plugin 機制整合到 AI agent 工作流

## 技術棧

| 類別 | 技術 | 版本 | 用途 |
|------|------|------|------|
| 主要格式 | Markdown | — | Skill 定義、文件 |
| YAML Frontmatter | YAML | — | Skill metadata（name、description） |
| 腳本語言 | Bash | 3.2+ | Session hook、SDD cache、idea-refine |
| Plugin 格式 | JSON | — | `.claude-plugin/plugin.json`、`marketplace.json` |
| CI/CD | GitHub Actions | — | Plugin 驗證與安裝測試 |
| Node.js 工具 | `@anthropic-ai/claude-code` | — | CI 驗證用 CLI |
| 依賴工具 | `jq`, `curl`, `shasum` | — | SDD cache hook 執行環境 |

## 目錄結構（3 層）

```
agent-skills/
├── skills/                            ← 21 個核心技能（每個含 SKILL.md）
│   ├── idea-refine/                   ← Define 階段：想法精煉
│   │   ├── SKILL.md
│   │   └── scripts/idea-refine.sh    ← 初始化 docs/ideas/ 目錄的腳本
│   ├── spec-driven-development/       ← Define 階段：規格先行
│   │   └── SKILL.md
│   ├── planning-and-task-breakdown/   ← Plan 階段
│   │   └── SKILL.md
│   ├── incremental-implementation/    ← Build 階段
│   ├── context-engineering/           ← Build 階段
│   ├── source-driven-development/     ← Build 階段（含 SDD cache hook）
│   ├── frontend-ui-engineering/       ← Build 階段
│   ├── test-driven-development/       ← Build 階段
│   ├── api-and-interface-design/      ← Build 階段
│   ├── browser-testing-with-devtools/ ← Verify 階段
│   ├── debugging-and-error-recovery/  ← Verify 階段
│   ├── code-review-and-quality/       ← Review 階段
│   ├── code-simplification/           ← Review 階段
│   ├── security-and-hardening/        ← Review 階段
│   ├── performance-optimization/      ← Review 階段
│   ├── git-workflow-and-versioning/   ← Ship 階段
│   ├── ci-cd-and-automation/          ← Ship 階段
│   ├── deprecation-and-migration/     ← Ship 階段
│   ├── documentation-and-adrs/        ← Ship 階段
│   ├── shipping-and-launch/           ← Ship 階段
│   └── using-agent-skills/            ← Meta skill（技能發現導覽）
├── agents/                            ← 3 個 specialist personas
│   ├── README.md                      ← 編排模式說明
│   ├── code-reviewer.md               ← Staff Engineer 角色
│   ├── security-auditor.md            ← Security Engineer 角色
│   └── test-engineer.md               ← QA Specialist 角色
├── hooks/                             ← Session lifecycle hooks
│   ├── hooks.json                     ← Hook 設定（SessionStart）
│   ├── session-start.sh               ← 注入 using-agent-skills meta-skill
│   ├── SDD-CACHE.md                   ← SDD cache hook 說明文件
│   ├── sdd-cache-pre.sh               ← PreToolUse WebFetch hook
│   ├── sdd-cache-post.sh              ← PostToolUse WebFetch hook
│   ├── SIMPLIFY-IGNORE.md             ← simplify-ignore hook 說明
│   ├── simplify-ignore.sh             ← 設定 simplify 豁免清單
│   └── simplify-ignore-test.sh        ← simplify-ignore 測試腳本
├── .claude/commands/                  ← 7 個 slash commands
│   ├── spec.md                        ← /spec → spec-driven-development
│   ├── plan.md                        ← /plan → planning-and-task-breakdown
│   ├── build.md                       ← /build → incremental-implementation + TDD
│   ├── test.md                        ← /test → test-driven-development
│   ├── review.md                      ← /review → code-review-and-quality
│   ├── code-simplify.md               ← /code-simplify → code-simplification
│   └── ship.md                        ← /ship → parallel fan-out + merge
├── .claude-plugin/                    ← Claude Code plugin 元資料
│   ├── plugin.json                    ← Plugin 名稱、版本、author
│   └── marketplace.json              ← Marketplace 安裝設定
├── references/                        ← 補充 checklist（技能使用時載入）
│   ├── testing-patterns.md
│   ├── security-checklist.md
│   ├── performance-checklist.md
│   ├── accessibility-checklist.md
│   └── orchestration-patterns.md
├── docs/                              ← 各工具安裝指南
│   ├── skill-anatomy.md               ← SKILL.md 格式規格（最重要文件）
│   ├── getting-started.md
│   ├── cursor-setup.md
│   ├── gemini-cli-setup.md
│   ├── windsurf-setup.md
│   ├── copilot-setup.md
│   └── opencode-setup.md
├── .github/workflows/
│   └── test-plugin-install.yml        ← CI：驗證 plugin 結構 + 安裝測試
├── CLAUDE.md                          ← Claude Code 專案說明
├── AGENTS.md                          ← OpenCode 整合說明
├── README.md                          ← 主文件（含全部技能說明）
└── CONTRIBUTING.md                    ← 貢獻指南
```

## 既有文件摘要

### 權威文件來源

| 文件 | 路徑 | 重要性 |
|------|------|--------|
| Skill 格式規格 | `docs/skill-anatomy.md` | ★★★ 最高 — 定義所有 SKILL.md 的規範 |
| Plugin metadata | `.claude-plugin/plugin.json` | ★★★ Plugin 名稱/版本/作者 |
| Marketplace 設定 | `.claude-plugin/marketplace.json` | ★★★ 安裝識別碼 |
| Meta-skill | `skills/using-agent-skills/SKILL.md` | ★★★ 技能發現導覽圖 |
| Hook 設定 | `hooks/hooks.json` | ★★ SessionStart 行為 |
| SDD cache 說明 | `hooks/SDD-CACHE.md` | ★★ cache 機制完整說明 |
| 編排模式 | `references/orchestration-patterns.md` | ★★ Multi-agent 設計規範 |
| Persona 說明 | `agents/README.md` | ★★ Persona/Skill/Command 三層關係 |

### 落差分析（既有文件 vs 實際程式碼）

1. **README.md 說 20 個 skills**，但 `skills/` 目錄實際有 21 個子目錄（含 `using-agent-skills`）。README 的技能清單未包含 `using-agent-skills`，但它確實是一個 SKILL.md。→ `skills/using-agent-skills/SKILL.md` 是 meta-skill，README 將其歸類為「meta」而非 20 個主要技能之一。
2. **文件說有 7 個 slash commands**，但 `.claude/commands/` 只有 7 個 `.md` 檔案（spec, plan, build, test, review, code-simplify, ship）—— 一致。
3. **AGENTS.md 格式規格**（OpenCode 格式，含 scripts/ 和 zip 包）與 `docs/skill-anatomy.md`（Claude Code 格式）描述有差異：AGENTS.md 要求 zip 包和腳本，Claude Code 格式只需要 SKILL.md。兩個格式都是有效的，針對不同 runtime。

## 統計摘要

| 項目 | 數量 |
|------|------|
| 總檔案數 | 65 |
| Markdown 文件數 | 53 |
| SKILL.md 檔案數 | 21 |
| Bash 腳本數 | 6 |
| Slash commands | 7 |
| Agent personas | 3 |
| Reference 文件 | 5 |
| 所有 SKILL.md 總行數 | ~5,962 |
