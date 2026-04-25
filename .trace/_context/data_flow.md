# Stage 2.2 Data Flow

本專案是文件型 plugin，「data flow」指的是「用戶發出指令」→「agent 執行 skill 工作流程」的完整路徑。

## 代表性 Use Case：`/build` 執行流程

選擇 `/build` 是因為它組合了兩個技能（incremental-implementation + TDD），展示了技能組合模式。

### 完整路徑

```
[用戶] 輸入: /build
    │
    ▼
[Claude Code] 查找 .claude/commands/build.md
    │
    │ 讀取內容:
    │   "Invoke the agent-skills:incremental-implementation skill
    │    alongside agent-skills:test-driven-development."
    ▼
[Agent] 載入兩個 SKILL.md 到 context:
    ├── skills/incremental-implementation/SKILL.md
    └── skills/test-driven-development/SKILL.md
    │
    ▼
[Agent] 執行 incremental-implementation 工作流:
    Step 1: 從計劃（tasks/todo.md）挑選下一個任務
    Step 2: 讀取任務的驗收標準
    Step 3: 載入相關程式碼 context
    Step 4: 進入 TDD 循環 ──────────────────┐
        4a. 寫失敗測試 (RED)                │
        4b. 實作最小程式碼 (GREEN)          │
        4c. 執行完整測試套件               │
        4d. 執行建置驗證                    │
        4e. commit                         │
        4f. 標記任務完成 ──────────────────┘
    │
    ▼
若任何步驟失敗:
    └── 呼叫 debugging-and-error-recovery 技能
```

### 資料轉換層

| 層 | 輸入 | 輸出 | 位置 |
|----|------|------|------|
| Routing | 用戶輸入 `/build` | command 內容 | `.claude/commands/build.md` |
| Skill Loading | command 內容 | SKILL.md 載入到 context | `skills/*/SKILL.md` |
| Task Selection | `tasks/todo.md` | 當前任務 + 驗收標準 | 用戶專案（不在 agent-skills repo）|
| Code Context | 當前任務 | 相關程式碼 | 用戶程式庫 |
| TDD Cycle | 任務規格 | 通過測試的實作 | 用戶程式庫 |
| Verification | 測試結果 + 建置輸出 | pass/fail 判斷 | 執行環境 |
| Artifact | pass 後的 commit | git history | 用戶程式庫 |

## `/ship` Fan-Out 資料流（多 agent 並行）

`/ship` 是最複雜的資料流，實作 parallel fan-out 模式：

```
[用戶] /ship
    │
    ▼
[Main Agent] 讀取 .claude/commands/ship.md
    │
    ├─── 同時啟動 3 個 Sub-agents ───────────────────┐
    │                                                 │
    ▼                    ▼                    ▼       │
[code-reviewer]  [security-auditor]   [test-engineer]│
    │                    │                    │       │
    │ 五軸審查            │ OWASP 安全審查      │ 測試覆蓋分析
    │ - Correctness      │ - Input handling   │ - 測試金字塔
    │ - Readability      │ - Auth/Authz       │ - 邊界案例
    │ - Architecture     │ - Data protection  │ - Prove-It
    │ - Security         │ - Infrastructure   │
    │ - Performance      │ - 3rd party        │
    │                    │                    │       │
    └────────────────────┴────────────────────┘       │
                         │                            │
                         ▼                            │
              [Main Agent] 合併報告 ──────────────────┘
                  1. 彙整 Critical/Important 發現
                  2. 安全問題升級為 launch blockers
                  3. Cross-reference 各 persona 發現
                  │
                  ▼
              輸出 go/no-go 決策 + rollback plan
```

## Skill 內部的資料流（以 spec-driven-development 為例）

```
[觸發] 用戶說「我要建立一個新功能」
    │
    ▼
PHASE 1: Specify
    輸入: 模糊需求
    處理: 詢問澄清問題（AskUserQuestion）
    輸出: 具體化需求

PHASE 2: Plan
    輸入: 具體需求
    處理: 識別依賴圖、拆分垂直切片
    輸出: SPEC.md（6 個區塊：目標、指令、結構、風格、測試、邊界）

PHASE 3: Tasks
    輸入: SPEC.md
    處理: 分解為可驗證任務
    輸出: tasks/plan.md + tasks/todo.md

PHASE 4: Implement
    輸入: tasks/todo.md
    處理: TDD 循環 × 每個任務
    輸出: 通過測試的實作 + commits
```

## SDD Cache Hook 資料流

`source-driven-development` 技能有一個旁路資料流：

```
agent 執行 WebFetch(url, prompt)
    │
    ▼
PreToolUse hook (sdd-cache-pre.sh)
    │
    ├── 計算 sha256(url) → CACHE_FILE
    │
    ├── 若 CACHE_FILE 不存在 → exit 0（允許 WebFetch 執行）
    │
    └── 若 CACHE_FILE 存在:
           發送 HEAD 請求（帶 If-None-Match / If-Modified-Since）
               │
               ├── 304 Not Modified → 返回快取內容，exit 2（攔截 WebFetch）
               └── 200 OK → 允許 WebFetch 重新執行
                      │
                      ▼
               PostToolUse hook (sdd-cache-post.sh)
                      │
                      └── 儲存 {url, prompt, etag, last_modified, content}
                          到 .claude/sdd-cache/<hash>.json
```

## 跨平台資料流差異

| 平台 | 入口 | 技能載入方式 | 狀態保存 |
|------|------|------------|---------|
| Claude Code | Plugin + slash commands | 自動（SessionStart hook） | SPEC.md, tasks/ |
| Cursor | `.cursor/rules/` | 靜態（always-on） | — |
| Gemini CLI | `gemini skills install` | 動態（按需） | — |
| OpenCode | AGENTS.md 意圖映射 | 自動（skill tool） | — |
| 手動 | 貼入 system prompt | 靜態 | — |
