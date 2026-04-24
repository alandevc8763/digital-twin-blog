---
title: "Agent Engineering Rigor"
date: 2026-04-24
categories: ["General"]
tags: ["General", "Agent-Engineering-Rigor.Md"]
draft: false
---

# 對抗不確定性：生產級 AI Agent 的「外科手術」與「量化評估」

**發佈日期**: 2026-04-22
**分類**: LLMOps / SRE / Agent-Engineering
**關鍵字**: Agent穩定性, Eval-Framework, Karpathy-Guidelines, 閉環演化

---

## 01. Demo 與產品之間的「死亡之谷」

在 AI 開發中，最危險的時刻就是當你對老闆或客戶展示一個「看起來很神奇」的 Demo 之後。

大多數 AI Agent 在 Demo 階段表現完美，是因為開發者在特定的路徑上進行了- **Prompt 過度擬合 (Over-fitting)**。但一旦進入生產環境，面對不可預測的用戶輸入，系統會迅速崩潰 $\rightarrow$ 出現無限循環、格式錯誤或邏輯幻覺。

**將 AI Agent 從 Demo 推向產品，需要的是 SRE (Site Reliability Engineering) 的嚴謹度，而非更多 Prompt 的疊加。**

---

## 02. 外科手術式修改 (Surgical Changes)

當 Agent 出現錯誤時，初學者傾向於「重新寫一遍 Prompt」或「增加更多指令」。這通常會導致 **「回歸失效 (Regression)」**：修好了 A Bug，卻引入了 B Bug。

我推崇 **「外科手術式修改」** 的原則：
1. **最小擾動**：僅觸碰必須修改的那一行代碼或 Prompt 片段。
2. **風格對齊**：嚴格匹配現有風格，不進行無關的重構。
3. **可追溯性**：每一處修改都必須直接對應到一個具體的失敗案例。

這就是為什麼在協作中，我們必須要求 LLM 停止「好心的重構」，專注於精確的修復。

---

## 03. 構建量化評估體系 (The Eval Framework)

如果你不能量化 Agent 的表現，你就無法優化它。生產級 Agent 必須擁有自己的 **Harness (測試框架)**。

### 1. 基準集 (Golden Dataset)
建立一個包含 50-100 個 $\text{輸入} \rightarrow \text{期望輸出}$ 的高質量測試集。每次更新 Prompt 後，必須全量跑一遍，確保準確率沒有下降。

### 2. LLM-as-a-Judge
利用更強的模型 (如 Claude 3.5 Opus) 作為裁判，根據 `Faithfulness` (忠實度) 與 `Answer Relevance` (相關度) 給予打分。

### 3. 軌跡追蹤 (Tracing)
使用 Langfuse 等工具記錄每一跳的 $\text{Prompt} \rightarrow \text{Tool Call} \rightarrow \text{Observation}$。當 Agent 失敗時，我們分析的是 **「軌跡 (Trace)」** 而非單純的結果。

---

## 04. 閉環演化：AI 系統的唯一生存方式

生產級 Agent 的生命週期應該是一個不斷縮小的負反饋環：
$\text{捕捉失敗案例} \rightarrow \text{分析 Trace 根因} \rightarrow \text{外科手術式修正} \rightarrow \text{回歸測試驗證} \rightarrow \text{部署}$

**不要追求一次性的完美，而要追求極速的演化。**

---

## 05. 結語

AI Agent 的工程化，本質上是在與「不確定性」作鬥爭。

當我們停止對 LLM 抱有「魔法」的幻想，開始用量化指標、回歸測試與嚴謹的修改路徑去對待它時，AI 才會真正成為一個可信賴的生產力工具。