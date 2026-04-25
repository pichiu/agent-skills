# Stage 1 線上搜尋結果

## 搜尋摘要

本專案為純文件型開源專案，無官方網站，但維護者 Addy Osmani 有個人部落格與 Substack，撰寫了多篇關於 agentic engineering 的文章。

## 主要資源連結

### 官方/半官方資源

| 資源 | 連結 | 摘要 |
|------|------|------|
| GitHub 主頁 | https://github.com/addyosmani/agent-skills | 22k+ stars，包含完整文件 |
| Claude Plugin Hub | https://www.claudepluginhub.com/marketplaces/addyosmani-addy-agent-skills | Plugin 市集頁面 |
| Claude Code Docs (Skills) | https://code.claude.com/docs/en/skills | 官方 Claude Code skills 說明 |
| Claude API Docs (Agent Skills) | https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview | Anthropic 官方 Agent Skills 概念說明 |

### 維護者部落格文章

| 標題 | 連結 | 重要性 |
|------|------|--------|
| Agentic Engineering | https://addyosmani.com/blog/agentic-engineering/ | ★★★ 介紹 agentic engineering 概念，專案背景思想 |
| Self-Improving Coding Agents | https://addyosmani.com/blog/self-improving-agents/ | ★★ 自我改進 agent 設計思路 |
| The future of agentic coding | https://addyosmani.com/blog/future-agentic-coding/ | ★★ conductors to orchestrators 演進 |
| How to write a good spec for AI agents | https://addyo.substack.com/p/how-to-write-a-good-spec-for-ai-agents | ★★★ spec-driven-development 的延伸說明 |

### 社群文章

| 標題 | 連結 | 摘要 |
|------|------|------|
| Agent Skills: Teaching AI agents to code like senior engineers | https://www.rushis.com/agent-skills-teaching-ai-agents-to-code-like-senior-engineers/ | 第三方解析文章 |
| DEV Community 介紹文 | https://dev.to/_46ea277e677b888e0cd13/agent-skills-19-production-grade-skills-that-make-ai-coding-agents-work-like-senior-engineers-5bi9 | 社群討論 |
| Threads 討論 | https://www.threads.com/@codeaholicguy/post/DWs2NI2DZmA/ | Addy Osmani 分享 19 個 skills 的討論串 |
| Understanding Claude Code: Skills vs Commands vs Subagents vs Plugins | https://www.youngleaders.tech/p/claude-skills-commands-subagents-plugins | ★★ 釐清概念差異的分析文章 |

### 相關生態系工具

| 工具 | 連結 | 與本專案關係 |
|------|------|-------------|
| awesome-agent-skills | https://github.com/VoltAgent/awesome-agent-skills | 社群策展 1000+ skills |
| claude-code-plugins-plus-skills | https://github.com/jeremylongshore/claude-code-plugins-plus-skills | 423 plugins，含本專案 |
| web-quality-skills | https://github.com/addyosmani/web-quality-skills | 同作者，針對 Web 品質的 skills |
| ali-claude-skills | https://github.com/alirezarezvani/claude-skills | 232+ skills 集合，兼容多平台 |
| Jimmy Song 中文摘要 | https://jimmysong.io/ai/addyosmani-agent-skills/ | 中文介紹文章 |

## 關鍵 Takeaway

1. **設計哲學**：AI coding agent 預設走最短路徑（跳過 spec、測試、安全審查）。本專案透過「強制執行工作流程」解決這個問題，而不是提供參考文件。

2. **Google Engineering 文化影響**：Skills 直接借鑑了 Google 的工程實踐——
   - Hyrum's Law（API 設計）
   - Beyonce Rule（測試）
   - 測試金字塔 80/15/5
   - Chesterton's Fence（簡化）
   - Trunk-based development（Git 工作流）
   - Shift Left + feature flags（CI/CD）

3. **多平台策略**：同一份 Markdown 技能文件，透過不同整合方式支援：
   - Claude Code（原生 Plugin + slash commands）
   - Cursor（`.cursor/rules/` 或 `.cursorrules`）
   - Gemini CLI（`gemini skills install`）
   - Windsurf（rules 設定）
   - GitHub Copilot（`.github/copilot-instructions.md`）
   - OpenCode（AGENTS.md + skill tool）

4. **GitHub 活躍度**：截至 2026-04，有多個 open issues（#77, #81, #83, #85, #89, #95, #97），最近的 PR 包含編排模式文件化（PR #86）、SDD cache hook（PR #80）等功能增強。

5. **Progressive disclosure 設計原則**：SKILL.md 是入口點，supporting files 按需載入，減少 token 消耗。skill description 欄位注入 system prompt，控制在 1024 字元內。

## 注意事項

- ⚠️ 未驗證：GitHub Issues 的確切內容（需要 GitHub API 存取）
- ⚠️ 未驗證：Claude Code Marketplace 上的評分/下載數
