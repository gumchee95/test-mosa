# MOSA Framework Core Rules (GEMINI.md) — v2.5
> **Last Updated**: 2026-04-17 | **Architecture**: MOSA

全局指令集。所有 Agent 必須嚴格遵守。

## 統一啟動序列 (強制，唯一)

所有任務啟動時，**Context Sniffing** 先於一切：
- 已載入（GEMINI.md + SKILL.md 在上下文中）→ 跳過 Steps 1-2，從 Step 3 開始。
- Naked Session（新對話/上下文丟失）→ 從 Step 1 開始。
- **拓撲感知 (Graph Topology)**：若檢測到 `{Workspace_Root}/graphify-out/GRAPH_REPORT.md`，強制要求任何 Agent 優先讀取此報告以獲取項目宏觀結構（God Nodes 與檔案依賴關係），嚴禁在已有圖譜的情況下使用全域 grep 探索架構。

**完整啟動序列（7 步）：**

| Step | 動作 | 責任方 |
|------|------|--------|
| 1 | 讀取 GEMINI.md（鎖定全局規則） | Host Agent |
| 2 | 讀取 auto-skill/SKILL.md（Meta-Logic + 核心循環） | Host Agent |
| 3 | 呼叫 orchestrator_agent（逆向工程 + 原子解構 + task.md 生成） | Host Agent |
| 4 | orchestrator_agent 呼叫 router_agent（技能檢索） | orchestrator |
| 5 | 載入 Execution Skill → 派發對應 Sub-Agent 執行 | orchestrator |
| 6 | audit_agent 審核（觸發條件見下方 §審計觸發規則） | orchestrator |
| 7 | 任務結束 → GC + auto-skill Step 5 經驗記錄詢問 | auto-skill |

> **注意**：Steps 1-2 為初始化（僅 Naked Session），Steps 3-7 為執行循環（每輪任務）。

## 審計觸發規則 (Step 6)

audit_agent 在以下情況**強制觸發**：
- 任務涉及 ≥5 個文件的寫入操作
- 任務標記為 [Critical] 或涉及金融/合規數據
- 用戶明確要求審核
- 連續 2 次 Sub-Agent 返回 [Status: Fail]

其餘場景 audit_agent 為可選。

## 輸出規範 (強制)

- 使用 Point form，每項 ≤10 詞。
- 嚴禁禮貌廢話，直接輸出結果。
- ff 模式：僅輸出改動部分，禁止全文複讀。
- Agent 間通訊允許 [Status] [Data] [Next_Step] 結構。

## 持久化與 GC (強制)

- 多步任務：在 01_Work/ 建立 session_state.json。
- Pointers Only：僅存路徑指針，嚴禁寫入全量數據。
- 任務完成：清空 session_state.json，轉移至 02_Output。

## 文件命名規範 (強制)

| 文件 | 用途 | 維護者 |
|------|------|--------|
| `task.md` | 任務規劃與進度追蹤（TODO list） | orchestrator_agent |
| `task_results.md` | Sub-Agent 執行結果指針匯總 | Execution Sub-Agent |
| `session_state.json` | 跨回合狀態指針（臨時） | orchestrator_agent |
| `prompt_stack.md` | 當前 Workspace 的長期記憶錨點 | mosa-harmonizer |

## Agent 管理工作流 (強制)

- 所有 Agent 定義統一存放：~/.gemini/antigravity/workflows
- 嚴禁存放於 skills/ 或項目本地 .agents/ 目錄。
- 維持 Single Source of Truth。

## 工作空間隔離 (強制)

- 從當前 Active Document 向上搜尋最近 00_System 作為 Workspace Root。
- 嚴禁跨越 Sibling 資料夾。
- 所有路徑使用 ~/ 相對路徑，禁止硬編碼絕對路徑。
- `prompt_stack.md` 位於：`{Workspace_Root}/00_System/prompt_stack.md`（明確路徑）。
- `state.json` 位於：`{Workspace_Root}/00_System/state.json`（含 turn_count 與 drift_threshold）。

## MOSA 架構規範 (強制)

- 技能：YAML Frontmatter Markdown，存入 ~/.gemini/antigravity/skills/
- Layer C Sub-Agent：純 SOP 執行機，不內建業務邏輯。
- 狀態傳遞：僅透過 session_state.json 或 task_results.md。
- 語言偏好：依用戶最近回覆或 prompt_stack.md 決定。

## 最高執行原則

- GEMINI.md 為憲法級規則，所有衝突以此為準。
- 任何修改需更新本檔案並記錄日期。
