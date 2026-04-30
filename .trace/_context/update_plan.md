# 更新計畫

更新日期：2026-04-30

## 變更摘要
- **Base Commit 範圍**: 1f66d57..19e49a0
- **變更檔案數**: 11（排除 README 微調後：10）
- **變更幅度**: ≈ 14%（中度更新）
- **判定**: 繼續增量更新

## 受影響文件與更新策略

### CODEBASE_MAP.md — 需要更新
- **原因**: 新增 `.gemini/commands/` 目錄（7 個 TOML）、`.opencode/skills` symlink
- **影響段落**:
  - Annotated directory tree（新增 `.gemini/` 和 `.opencode/` 兩個頂層目錄）
  - 「我想改 X 要看哪裡？」速查表（新增 Gemini CLI 指令相關條目）
  - Mermaid 模組依賴圖（可不動，結構未改變）
- **更新策略**: Main Agent（局部追加兩個目錄段落）

### ARCHITECTURE.md — 需要更新
- **原因**:
  1. Gemini CLI 現在有原生 slash commands（`.gemini/commands/`），與 Claude Code 並列
  2. `.opencode/skills` symlink 改變了 OpenCode 整合方式
  3. `session-start.sh` bug fix（jq 依賴 + JSON escape 修正）
- **影響段落**:
  - 組件清單（新增 `.gemini/commands/` 組件）
  - 通訊模式（Gemini CLI slash commands 新增）
  - Hook injection 說明（session-start.sh 新增 jq fallback）
- **更新策略**: Main Agent（局部追加，不需大幅改寫）

### API_SURFACE.md — 需要更新
- **原因**: Gemini CLI 新增 7 個 TOML slash commands，介面略有差異（`/planning` vs `/plan`、TOML 格式）
- **影響段落**:
  - Slash Commands 完整參考（新增 Gemini CLI 欄位）
  - 多平台安裝指令對照表（Gemini CLI slash commands 支援狀態從 ❌ 改為 ✅）
- **更新策略**: Main Agent（新增 Gemini CLI commands 對照段落）

### DEV_GUIDE.md — 需要更新
- **原因**:
  1. `session-start.sh` 現在依賴 jq（新增 prerequisite）
  2. Gemini CLI 安裝現在有 slash commands（新增說明）
  3. `.opencode/skills` symlink 簡化 OpenCode 使用方式
  4. Debugging：jq 缺失時的 session-start 失敗訊息改變
- **影響段落**:
  - Prerequisites（新增 jq 依賴說明）
  - Debugging 技巧（session-start hook 的錯誤訊息更新）
- **更新策略**: Main Agent（局部修改 Prerequisites + Debugging 段落）

### INDEX.md — 輕量更新
- **原因**: 新增 `.gemini/commands/` 代表 Gemini CLI 整合升級，可在技術棧表格加一行
- **影響段落**: 關鍵指令速查（Gemini CLI slash commands 新增）
- **更新策略**: Main Agent（加一行）

### DATA_MODEL.md — 不需更新
- **原因**: 無資料結構變更（TOML 格式是新的 command format，但不在 DATA_MODEL 的涵蓋範圍）

### DISCOVERY_LOG.md — 需要更新
- **原因**: 追加本次更新的發現（JSON injection bug、jq 依賴、Gemini CLI 命名差異）
- **更新策略**: Main Agent（追加段落）

## 執行順序

1. CODEBASE_MAP.md（目錄結構基礎）
2. INDEX.md（技術棧速查）
3. ARCHITECTURE.md（架構理解）
4. API_SURFACE.md（介面參考）
5. DEV_GUIDE.md（開發指南）
6. DISCOVERY_LOG.md（探索紀錄，最後）
