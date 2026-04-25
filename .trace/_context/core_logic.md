# Stage 2.3 核心領域邏輯

## 核心 Abstraction：Skill

Skill 是本專案的「心臟」——一個結構化的工程工作流程，以 Markdown + YAML frontmatter 表達。

### Skill 的本質定義

從 `docs/skill-anatomy.md` 提取：

> **Skills are workflows agents follow, not reference docs they read.**
> Each has steps, checkpoints, and exit criteria.

關鍵設計原則（優先順序）：
1. **Process over knowledge**：步驟，不是事實
2. **Specific over general**：`run npm test` 而非「確認測試」
3. **Evidence over assumption**：每個驗證 checkbox 都需要可見的証據
4. **Anti-rationalization**：每個可能被跳過的步驟都要有反駁理由
5. **Progressive disclosure**：SKILL.md 是入口，supporting files 按需載入

### SKILL.md 強制格式

**YAML Frontmatter（必填）**（`docs/skill-anatomy.md`）：
```yaml
---
name: skill-name-with-hyphens  # 必須與目錄名稱一致
description: Guides agents through [task]. Use when [trigger]. # ≤1024 字元
---
```

`description` 的作用：注入到 agent 的 system prompt 作為 skill 發現機制的基礎。**不應包含流程步驟**，否則 agent 可能執行摘要版本而跳過完整 SKILL.md。

**標準段落結構（建議）**：
```
# Skill Title
## Overview          → 1-2 句說明技能做什麼及為什麼重要
## When to Use       → 觸發條件 + 排除條件
## Core Process      → 主要工作流程（核心）
## Common Rationalizations → Excuse + 反駁（關鍵差異化特性）
## Red Flags         → 違反技能的警告信號
## Verification      → 完成標準 checklist（需要可見証據）
```

## 核心 Pattern 分析

### Pattern 1：Gated Workflow（防跳關）

`spec-driven-development` (`skills/spec-driven-development/SKILL.md`) 實作了最嚴格的 gated workflow：

```
SPECIFY ──→ PLAN ──→ TASKS ──→ IMPLEMENT
   │          │        │          │
   ▼          ▼        ▼          ▼
 Human      Human    Human      Human
 reviews    reviews  reviews    reviews
```

設計意圖：強制每個階段有人類確認，防止 AI 自作主張推進。

### Pattern 2：TDD Loop（Red-Green-Refactor）

`test-driven-development` 和 `incremental-implementation` 共享 TDD 循環：

```
┌──────────────────────────────────────┐
│  Implement ──→ Test ──→ Verify ──┐   │
│      ▲                           │   │
│      └───── Commit ◄─────────────┘   │
│             │                        │
│             ▼                        │
│         Next slice                   │
└──────────────────────────────────────┘
```

`incremental-implementation/SKILL.md` 說明：「每次不超過 ~100 行」，「垂直切片優先」。

### Pattern 3：Anti-Rationalization Table（防口號化）

每個 skill 都包含「Common Rationalizations」表格，這是本專案最獨特的設計：

```markdown
| Rationalization | Reality |
|----------------|---------|
| "I'll add tests later" | Tests written after the fact test what the code does, not what it should do |
| "This is too small for a spec" | 50% of bugs come from informal "obvious" changes |
```

這個設計應對了「AI agent 自我合理化跳過步驟」的核心問題。

### Pattern 4：Five-Axis Review（多維審查）

`code-review-and-quality` 和 `code-reviewer` agent 共享五軸模型：

```
      Correctness
           │
Readability ── Architecture
           │
    Security ── Performance
```

每個軸都有具體的 checklist 而非模糊描述。

### Pattern 5：Parallel Fan-Out + Merge

`/ship` 指令實作的編排模式（`references/orchestration-patterns.md` Pattern 3）：

```
fan out: 3 並行 sub-agents（不同觀點）
merge:   main agent 合併（小型 merge step 留在 main context）
output:  單一 go/no-go 決策
```

關鍵限制：sub-agents 不能呼叫其他 sub-agents（Claude Code 平台限制），personas 不做相互呼叫（架構原則）。

### Pattern 6：Skill Discovery Flowchart（meta-skill）

`using-agent-skills` 是 meta-skill，實現技能的「路由」：

```
Task arrives
    │
    ├── Vague idea? ──────────→ idea-refine
    ├── New project/feature? ──→ spec-driven-development
    ├── Have a spec? ──────────→ planning-and-task-breakdown
    ├── Implementing? ─────────→ incremental-implementation
    │   ├── UI? ──────────────→ frontend-ui-engineering
    │   └── API? ─────────────→ api-and-interface-design
    ├── Testing? ──────────────→ test-driven-development
    ├── Something broke? ──────→ debugging-and-error-recovery
    ├── Reviewing? ────────────→ code-review-and-quality
    └── Deploying? ────────────→ shipping-and-launch
```

這個 flowchart 透過 SessionStart hook 注入到每個 session。

## 「心臟」技能：using-agent-skills

這個 meta-skill 是整個系統的控制中心，因為它：
1. 透過 SessionStart hook 強制注入每個 session
2. 定義了 6 個「Core Operating Behaviors」（Surface Assumptions、Manage Confusion Actively、Push Back When Warranted、Enforce Simplicity、Maintain Scope Discipline、Verify Don't Assume）
3. 定義了 10 個「Failure Modes to Avoid」
4. 提供完整的技能生命週期序列

路徑：`skills/using-agent-skills/SKILL.md`

## Google Engineering DNA

從 `README.md` 中識別的 Google 工程實踐引用：

| Google 概念 | 出現在哪個 Skill |
|------------|----------------|
| Hyrum's Law | api-and-interface-design |
| One-Version Rule | api-and-interface-design |
| Beyonce Rule（你放進去的，你得維護） | test-driven-development |
| 測試金字塔 80/15/5 | test-driven-development |
| Chesterton's Fence | code-simplification |
| Trunk-based development | git-workflow-and-versioning |
| Change sizing ~100 lines | code-review-and-quality, git-workflow-and-versioning |
| Shift Left | ci-cd-and-automation |
| Code as liability | deprecation-and-migration |
| Feature flags | ci-cd-and-automation, shipping-and-launch |

參考來源：Software Engineering at Google + Google engineering practices guide

## 特殊技能：idea-refine

`idea-refine` 是唯一有 `scripts/` 子目錄的技能（`skills/idea-refine/scripts/idea-refine.sh`）。腳本功能很簡單——只是建立 `docs/ideas/` 目錄。真正的邏輯在 SKILL.md 中的 3 階段發散/收斂思考流程：

1. Understand & Expand（發散）：重述想法、提問、產出 5-8 個變體
2. Evaluate & Converge（評估）：壓力測試、暴露假設
3. Sharpen & Ship：輸出一頁 Markdown（問題陳述、建議方向、MVP 範圍、不做清單）

這個技能明顯受到 Apple 設計哲學影響（「簡單是終極複雜」、「從用戶體驗反推技術」等引言直接在 SKILL.md 中引用）。
