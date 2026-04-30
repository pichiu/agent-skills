# DISCOVERY_LOG.md — 探索紀錄

> 建立日期：2026-04-25
> 探索範圍：agent-skills 完整儲存庫（65 個檔案、21 個 SKILL.md）
> 探索方式：靜態分析 + Web 搜尋 + 程式碼對照

---

## 1. Web Search 發現摘要

### 主要資源連結與 Takeaway

| 資源 | 連結 | 關鍵 Takeaway |
|------|------|--------------|
| GitHub 主頁 | https://github.com/addyosmani/agent-skills | 22k+ stars，MIT 授權，純文件型 plugin |
| Claude Plugin Hub | https://www.claudepluginhub.com/marketplaces/addyosmani-addy-agent-skills | Marketplace 頁面，安裝識別碼 `addy-agent-skills` |
| Claude Code Docs (Skills) | https://code.claude.com/docs/en/skills | 官方 skill 格式規範，`description` ≤1024 字元要求 |
| Anthropic Agent Skills 概覽 | https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview | Plugin 協定完整說明 |
| awesome-agent-skills | https://github.com/VoltAgent/awesome-agent-skills | 社群策展 1000+ skills，本專案是重要入口 |
| Skills vs Commands vs Subagents | https://www.youngleaders.tech/p/claude-skills-commands-subagents-plugins | 釐清四個概念的差異（推薦閱讀） |

### Addy Osmani 部落格文章核心觀點摘要

**[Agentic Engineering](https://addyosmani.com/blog/agentic-engineering/)** ★★★
- AI agent 預設走最短路徑：跳過 spec、測試、安全審查
- 解法不是提供參考文件，而是「強制執行工作流程」（skills = process enforcement）
- 工程紀律需要被編碼進 agent 的指令中，而非留給 agent 自行判斷

**[How to write a good spec for AI agents](https://addyo.substack.com/p/how-to-write-a-good-spec-for-ai-agents)** ★★★
- spec-driven-development 的延伸說明：6 個區塊（目標、指令、結構、風格、測試、邊界）
- Spec 是 AI agent 的「有損壓縮合約」，模糊的 spec 直接導致幻覺式實作
- 建議在 spec 中明確列出「不做什麼（Out of scope）」

**[Self-Improving Coding Agents](https://addyosmani.com/blog/self-improving-agents/)** ★★
- Skills 本身是 self-improving 機制的基礎——agent 遵循 skill，skill 隨實踐迭代
- Progressive disclosure 設計：skill description 是誘餌，SKILL.md 是完整流程

**[The future of agentic coding](https://addyosmani.com/blog/future-agentic-coding/)** ★★
- 從「conductors（指揮人類）」到「orchestrators（協調 agents）」的角色轉變
- `/ship` 的 parallel fan-out 模式是這個演進的具體實作

---

## 2. 既有文件與程式碼的落差清單

- [ ] **落差 1**：README.md 說「All 20 Skills」，`skills/` 目錄實際有 **21 個子目錄**（含 `using-agent-skills`）
  - 文件說：「Under the hood, they activate these 20 skills」
  - 程式碼實際是：21 個 SKILL.md，`using-agent-skills` 是完整的 meta-skill，有 YAML frontmatter
  - 根本原因：README 將 `using-agent-skills` 歸為 meta 而非 20 個主要技能之一，但未明確說明排除邏輯

- [ ] **落差 2**：`using-agent-skills/SKILL.md` 的 flowchart 缺少 2 個已存在的技能
  - 文件說：flowchart 涵蓋所有技能
  - 程式碼實際是：`code-simplification` 和 `deprecation-and-migration` **未出現在** flowchart 的任何分支，也未出現在 Quick Reference 表格
  - 影響：agent 透過 SessionStart hook 注入 meta-skill 後，不會主動發現這兩個技能

- [ ] **落差 3**：AGENTS.md（OpenCode 格式）與 `docs/skill-anatomy.md`（Claude Code 格式）格式規格存在歧異
  - 文件說（AGENTS.md）：OpenCode skills 需要 `scripts/` 子目錄 + `.zip` 打包發布
  - 程式碼實際是：只有 `idea-refine` 有 `scripts/` 目錄，其他技能均無；沒有任何 `.zip` 檔案存在
  - 根本原因：OpenCode 格式是「建議規格」而非「現行實作」，兩份文件針對不同 runtime

- [ ] **落差 4**：CI 工作流未驗證 OpenCode zip 格式
  - 文件說（AGENTS.md）：需要 `.zip` 打包
  - 程式碼實際是：`.github/workflows/test-plugin-install.yml` 只驗證 Claude Code plugin 格式，zip 打包未在 CI 中測試

---

## 3. 程式碼中的 TODO/FIXME/HACK 彙整

> grep 搜尋範圍：所有 `.md`、`.sh`、`.json` 檔案（排除 `.trace/` 目錄）

**搜尋結果：無真正的 TODO/FIXME/HACK 標記**

程式碼中出現的 `TODO` 均為**教學範例**（非待辦事項）：

- `skills/documentation-and-adrs/SKILL.md:120-121`：展示「不應留 TODO comment」的反例
  ```
  // Don't leave TODO comments for things you should just do now
  // TODO: add error handling  ← Just add it
  ```
- `skills/documentation-and-adrs/SKILL.md:265`：「TODO comments that have been there for weeks」是 Red Flags 清單的一項
- `skills/shipping-and-launch/SKILL.md:28`：`- [ ] No TODO comments that should be resolved before launch` 是 launch checklist 的一個驗證項目

**結論**：
- [x] 無真實的技術債 TODO/FIXME 標記殘留在生產程式碼中
- [x] ⚠️ 未驗證：Bash 腳本內部是否有內嵌的改進提示（僅對 SKILL.md 做了掃描）

---

## 4. 未解答的疑問

### 功能行為類

- [ ] **SDD cache 的 curl 超時行為**：`hooks/sdd-cache-pre.sh` 使用 `curl --max-time 5`，但若網路完全不通（timeout 而非 refuse），hook 是否能在 5 秒內乾淨退出並回傳 `exit 0`？⚠️ 未驗證完整行為

- [ ] **SDD cache 並發行為**：多個並行 WebFetch 同時寫入 `.claude/sdd-cache/` 時，是否存在 race condition（多個腳本同時寫入同一 hash 的 JSON 檔案）？⚠️ 未驗證

- [ ] **Hook 在不同 Claude Code 版本的兼容性**：`exit 2`（攔截工具調用）的語意是否在所有 Claude Code 版本穩定？老版本是否支援？⚠️ 未驗證

### 平台整合類

- [ ] **Kiro IDE 整合的詳細設定**：`docs/` 中有 `opencode-setup.md` 但沒有 `kiro-setup.md`，README 只提到 `.kiro/skills/` 目錄。Kiro IDE 的 skill 自動觸發機制是否與 Claude Code 一致？⚠️ 未驗證

- [ ] **各平台的 Skills 自動觸發機制差異**：
  - Claude Code：SessionStart hook → meta-skill 注入 → flowchart 驅動
  - Gemini CLI：`gemini skills install` 後的觸發機制不明確
  - Cursor：靜態 always-on，無動態觸發，效果如何？⚠️ 未驗證

- [ ] **Agent Teams（實驗性功能）的具體啟用條件**：`agents/README.md` 提到 agent teams 是「experimental」，但未說明確切的啟用條件或 Claude Code 版本要求。⚠️ 未驗證

### 規格類

- [ ] **description 欄位的 1024 字元限制來源**：`docs/skill-anatomy.md` 說明此為 system prompt 注入的限制，但未說明這是 Claude Code platform 硬限制還是建議值。若超過是靜默截斷還是報錯？⚠️ 未驗證

---

## 5. 已知技術債

### 文件同步類

- [ ] **`using-agent-skills/SKILL.md` flowchart 缺少 2 個技能**
  - `code-simplification` 未出現在 flowchart 任何分支
  - `deprecation-and-migration` 未出現在 flowchart 任何分支
  - Quick Reference 表格（第 155-175 行）同樣未包含這兩個技能
  - 優先級：高（影響 agent 的技能發現）

- [ ] **README 計數錯誤**：README 說「All 20 Skills」，實際有 21 個 SKILL.md（含 meta-skill）
  - 應修正為「20 skills + 1 meta-skill」或「21 skills（含 meta-skill）」並加說明

- [ ] **AGENTS.md 與 `docs/skill-anatomy.md` 格式歧異**
  - 兩份規格文件描述不同 runtime 的格式，但沒有明確的「統一規格索引」
  - 新貢獻者容易混淆應遵循哪份規格

### 驗證缺口類

- [ ] **OpenCode zip 格式未在 CI 中驗證**
  - CI 只驗證 Claude Code plugin 格式（`claude plugin validate .`）
  - OpenCode 的 zip 打包流程無自動化驗證

- [ ] **各 SKILL.md 的 description 長度未在 CI 中強制驗證**
  - 雖然規格說 ≤1024 字元，但 CI 工作流只驗證 plugin 安裝，不驗證 description 長度
  - 新增技能時可能無意中超過限制

---

## 6. 需要更深入調查的區域

- [ ] ⚠️ 未驗證 — **各 SKILL.md 的實際內容品質評估**：21 個技能的「Common Rationalizations」表格和「Verification」checklist 的品質是否一致？部分較新的技能（如 `idea-refine`、`browser-testing-with-devtools`）是否符合 `docs/skill-anatomy.md` 的完整規格？

- [ ] ⚠️ 未驗證 — **SDD cache 在大量並發 WebFetch 時的行為**：`sdd-cache-post.sh` 的 async 寫入在高並發下的 race condition 未被文件提及，也未見 file locking 機制

- [ ] ⚠️ 未驗證 — **Hook 跨版本兼容性**：`hooks/hooks.json` 使用的 `$CLAUDE_PLUGIN_ROOT` 環境變數是否在所有 Claude Code 版本（包含本地開發用的 `--plugin-dir` 模式）都能正確解析

- [ ] ⚠️ 未驗證 — **Gemini CLI 技能自動觸發機制**：`gemini skills install` 安裝後，agent 如何決定何時啟用哪個 skill？是否有類似 SessionStart hook 的機制？

- [ ] ⚠️ 未驗證 — **`simplify-ignore` hook 在實際場景的效果**：`hooks/simplify-ignore-test.sh` 存在，但其實際攔截邏輯與 `code-simplification` 技能的互動方式未被詳細記錄

---

## 7. 建議與維護者/社群確認的問題

### 格式與規格

- [ ] **格式不一致問題是否有計劃統一**？
  - AGENTS.md（OpenCode 格式）vs `docs/skill-anatomy.md`（Claude Code 格式）描述的是不同 runtime 的規格，但缺乏明確的「多平台格式索引」
  - 建議：在 `docs/skill-anatomy.md` 頂部加一節「Cross-platform format differences」，指向各平台的差異說明

- [ ] **是否有計劃自動同步 `using-agent-skills` flowchart**？
  - 目前新增技能後需手動更新 `using-agent-skills/SKILL.md` 的 flowchart
  - `code-simplification` 和 `deprecation-and-migration` 目前已缺席
  - 建議：在 CONTRIBUTING.md 中明確要求「新增技能必須同步更新 flowchart」作為 PR checklist 項目

### 計數與命名

- [ ] **README「20 Skills」的數字是否應更新為 21**？
  - `using-agent-skills` 是完整的 SKILL.md，不應被靜默排除在技能計數之外
  - 或者應在 README 明確說明「20 個工程技能 + 1 個 meta-skill」的分類邏輯

### 測試覆蓋

- [ ] **CI 是否有計劃擴展到覆蓋 OpenCode zip 格式驗證**？
  - 目前 CI 只驗證 Claude Code plugin 格式
  - OpenCode 整合是明確支援的平台，應有對應的 CI 驗證

- [ ] **是否考慮加入 description 長度的 CI 驗證**（linting SKILL.md frontmatter）？
  - 可用簡單的 Bash/jq 腳本在 CI 中驗證所有 SKILL.md 的 description ≤1024 字元

---

## 增量更新記錄：1f66d57..19e49a0（2026-04-30）

### 本次 Upstream 變更摘要

5 個 commit，11 個檔案，包含 3 項主要功能：

1. **Gemini CLI 原生 slash commands**（`54cc926`、`e6d4005`）：新增 `.gemini/commands/` 目錄，7 個 TOML 格式指令。值得注意：`/planning` 而非 `/plan`，因 `/plan` 與 Gemini CLI 內建指令衝突。
2. **OpenCode symlink**（`36d26a6`）：`.opencode/skills -> ../skills/`，讓 OpenCode 不再需要手動設定路徑即可自動發現技能。
3. **session-start.sh Bug Fix**（`43a0dde`、`501d226`）：修復了兩個潛在問題——JSON injection（SKILL.md 含特殊字元時 heredoc 會產生無效 JSON）以及 jq 缺失時的 graceful fallback。

### 新發現

- [ ] **`/planning` 命名差異需要使用者注意**：Gemini CLI 用戶習慣 `/plan` 可能因找不到指令感到困惑。`docs/gemini-cli-setup.md` 已有說明，但其他地方（如 README 的指令速查表）可考慮加注釋。
- [ ] **jq 現在是 session-start hook 的必要依賴**：之前文件中 jq 只被標記為 SDD cache 的選用依賴。此次修改讓 jq 成為 meta-skill 注入的必要條件（雖然缺少時有 graceful fallback）。建議在 README 的 Prerequisites 中明確標注。
- [x] **OpenCode skills symlink 解決了之前的路徑問題**：之前 `AGENTS.md` 說明 OpenCode 透過意圖映射使用技能，實際路徑設定不清楚。現在 symlink 提供了明確的整合機制。

### 本次未受影響的文件

- `DATA_MODEL.md`：無資料結構變更
- `_context/recon.md`、`entry_points.md`、`core_logic.md`、`extensions.md`、`integrations.md`、`configuration.md`：基礎 context 未受影響

---

*探索紀錄截止日期：2026-04-30 | 探索者：AI 自動化分析*
