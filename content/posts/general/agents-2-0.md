---
title: "Agents 2 0"
date: 2026-04-24
categories: ["General"]
tags: ["General", "Agents-2-0.Md"]
draft: false
---

# 停止對 Prompt 的盲目猜測：Agents 2.0 與「符號學習」的演進

如果你曾經花過數小時在調整 Prompt，試圖透過增加「請一步步思考」或「你是一個專業的專家」來提升 Agent 的表現，那麼你應該意識到：**Prompt Engineering 本質上是在用手工方式「修改神經網絡的權重」。**

這種做法極其低效且不可擴展。真正的 Agent 進化，不應該依賴於人類的靈感，而應該依賴於系統性的**反饋循環**。這就是為什麼我最近在研究 **Agents 2.0 (aiwaves-cn)** 提出的「符號學習 (Symbolic Learning)」範式。

## 從「魔法」轉向「工程」：將 Agent 視為計算圖

Agents 2.0 最核心的洞察在於：**它把整個 Agent 的管線 (Pipeline) 視為一個計算圖 (Computational Graph)，而將 Prompt 和工具 (Tools) 視為圖中的「權重」。**

在傳統的 LLM 應用中，我們把模型當成黑盒。但在 Agents 2.0 的邏輯裡，Agent 的執行路徑就是一次 **Forward Pass (前向傳播)**。

當 Agent 執行失敗或結果不理想時，它會產生一種「語言損失 (Language Loss)」。接著，系統會啟動 **Language-Based Backpropagation (基於語言的反向傳播)**：
1. **反思 (Reflection)**：分析執行軌跡，找出導致失敗的關鍵節點。
2. **梯度更新 (Gradient Update)**：將反思結果轉化為對 Prompt 或工具選擇邏輯的修改。
3. **結構演化 (Self-Evolution)**：自動更新計算圖的拓撲結構，優化工具調用的順序。

這意味著 Agent 不再僅僅是「執行指令」，而是在不斷地**自我優化**。

## 為什麼這很重要？

目前的 Agent 框架大多專注於「編排 (Orchestration)」，即告訴 Agent 怎麼走。但 Agents 2.0 關注的是「演化 (Evolution)」，即讓 Agent 自己發現怎麼走最快。

對於生產級應用來說，這解決了兩個核心痛點：
- **不可預測性**：透過量化的軌跡分析 (Trajectory Analysis)，我們能清楚知道 Agent 在哪一步出了錯，而不是面對一個模糊的錯誤輸出。
- **維護成本**：當業務邏輯改變時，系統可以透過數據驅動的方式自動調整 Prompt，而非由工程師手動逐條修改。

## 結語：數據中心 AI 的回歸

Agents 2.0 的實踐證明了一件事：**AI 的下一個突破口不在於模型參數的增加，而是在於數據流的閉環。**

當我們停止對 LLM 抱有「魔法」的幻想，開始將其視為一個可以被量化、被反饋、被迭代的工程系統時，我們才真正進入了 Agentic Workflow 的 2.0 時代。

---
**技術關鍵字：**
- Symbolic Learning (符號學習)
- Language-Based Backpropagation (語言反向傳播)
- Self-Evolving Agents (自演化 Agent)