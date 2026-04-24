---
title: "Copilotkit Agent Ui"
date: 2026-04-24
categories: ["General"]
tags: ["General", "Copilotkit-Agent-Ui.Md"]
draft: false
---

# 擺脫對話框的束縛：為什麼 Agent-Native UI 是 AI 應用的下一站

很多人在開發 AI Agent 時，最直覺的思考方式就是「做一個對話框」。用戶輸入 $
ightarrow$ Agent 思考 $
ightarrow$ 回傳文字。

但如果你真的試著在生產環境運行 Agent，你會發現對話框其實是個巨大的瓶頸。當 Agent 需要處理複雜的狀態、更新數據或請求用戶確認時，純文字的互動會變得異常低效。這就是為什麼我最近在關注 **CopilotKit**，以及它背後推動的 **AG-UI Protocol**。

## 對話框之後的思考：什麼是 Agent-Native UI？

簡單來說，Agent-Native UI 意味著介面不再是靜態的，而是**隨 Agent 的計畫而動態演化**。

想像一下，當你要求 Agent 「幫我分析上季度的銷售數據並調整預算」時，它不應該只是回覆你「我正在分析中...」，而應該直接在介面上渲染出一個實時更新的分析圖表，並在需要你確認預算金額時，自動彈出一個輸入表單。

這就是 CopilotKit 核心解決的問題。它不再把 UI 當成單純的輸出端，而是將其視為 Agent 執行路徑的一部分。

## CopilotKit 的技術核心

CopilotKit 其實是為前端開發者提供了一套工具鏈，讓 AI Agent 能夠直接「操縱」前端介面：

1. **Generative UI (生成式 UI)**：Agent 可以根據當前的狀態，在運行時決定要渲染哪個組件。這讓 UI 從「預定義」變成了「動態生成」。
2. **共享狀態層 (Shared State Layer)**：這是最關鍵的部分。後端 Agent 的狀態與前端 UI 實時同步。不用再寫複雜的 Polling 或 WebSocket 邏輯，Agent 修改狀態，UI 直接反應。
3. **內建的 HITL (Human-in-the-Loop)**：在生產級 Agent 中，完全自動化是危險的。CopilotKit 提供了一套標準流程，讓 Agent 能在關鍵節點「暫停」，請求用戶確認或修改，然後再繼續執行。
4. **`useAgent` Hook**：對於 React 開發者來說，這就像是一個控制開關，讓前端能以程式化方式讀取 Agent 的狀態或觸發動作。

## 從「工具調用」到「介面同步」

目前的 Agent 框架（如 LangGraph 或 CrewAI）大多專注於後端的邏輯編排（Orchestration）。但後端跑得再快，如果前端還在等文字流 (Streaming) 輸出，用戶體驗依然是破碎的。

CopilotKit 的價值在於它填補了這個 **Interaction Layer**。它讓後端 Tool 的回傳結果不再僅僅是 `string`，而可以是 `Frontend Component`。

## 結語

AI 應用的演進路徑很清晰：從 **Chatbot** $
ightarrow$ **Copilot** $
ightarrow$ **Agent-Native App**。

我們正處在從「跟 AI 聊天」轉向「與 AI 協作於同一介面」的轉折點。當 UI 能夠隨著 Agent 的意圖而流動時，AI 才會真正從一個「插件」變成應用的核心。

---
**參考資源：**
- GitHub: [CopilotKit/CopilotKit](https://github.com/CopilotKit/CopilotKit)
- 協作框架: LangGraph, CrewAI