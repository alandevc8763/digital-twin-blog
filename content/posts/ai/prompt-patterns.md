---
title: "Prompt Patterns"
date: 2026-04-24
categories: ["AI"]
tags: ["AI", "Ai-Engineering", "Prompt-Patterns.Md"]
draft: false
---

# 突破 Prompt 的迷思：從「咒語」轉向「工程模式」
在 AI 圈子裡，很多人把 Prompt Engineering 描述成一種「煉金術」或「寫咒語」。他們會分享一些神奇的短語，比如「這對我的職業生涯至關重要」或者「深呼吸，一步步思考」，並宣稱這能神奇地提升模型表現。
但如果你試過將這些「咒語」部署到生產環境，你很快會發現：**咒語是不可靠的。**
同樣的 Prompt 在不同版本的模型上表現迥異，甚至在相同的模型上，只要輸入數據稍微變動，輸出結果就會從「完美」跌落到「胡言亂語」。
對於工程師來說，我們不能依賴隨機性。Prompt Engineering 的本質不應該是「尋找正確的詞彙」，而應該是 **「透過結構化約束，縮小模型的搜索空間」**。
## Prompt 的本質：縮小搜索空間
LLM 的運作邏輯是預測下一個 token。如果你給的指令太模糊，模型在生成答案時，其可能的路徑（搜索空間）會非常巨大，這就導致了不穩定性。
**「咒語」之所以有效，是因為它們在潛意識中誘導模型進入某個特定的概率分佈。** 但更專業的做法是提供**結構化約束**。
### 1. 從「指令」轉向「工作記憶」 (CoT)
很多人知道 `Think step-by-step` 有用，但不知道為什麼。
從底層來看，這是在強迫模型將「推理過程」顯式化地寫在輸出中。這些生成的推理 token 實際上變成了模型的 **「外部工作記憶」**。模型在生成最終答案前，先在上下文（Context）中建立了一條邏輯鏈條，這大大降低了邏輯跳躍導致的錯誤。
**工程實踐**：不要只要求模型「思考」，而要給它 **Few-Shot CoT**。提供 3-5 個 `問題 $\rightarrow$ 推理過程 $\rightarrow$ 答案` 的範例，直接定義它的思考路徑。
### 2. 拒絕「上帝 Prompt」：模組化分解
很多開發者的習慣是寫一個幾千字的「上帝 Prompt」，把所有規則、背景、格式要求全部塞進去。
這會導致兩個問題：
- **Lost-in-the-Middle**：模型會忽略掉 Prompt 中間部分的指令。
- **指令衝突**：當規則太多時，模型在權衡不同指令的優先級時會產生混亂。
**工程實踐**：採取 **Prompt Decomposition (分解)**。
將一個複雜任務拆成流水線：
`提取實體 (Prompt A)` $\rightarrow$ `分析關係 (Prompt B)` $\rightarrow$ `合成報告 (Prompt C)`。
這樣你面對的是三個簡單、確定且易於評測的模組，而不是一個不可控的黑盒子。
### 3. 建立結構化錨點 (Structural Anchors)
不要用 `請總結以下文本：[文本]` 這種模糊的寫法。在處理大規模數據時，模型很容易分不清哪裡是「指令」，哪裡是「待處理數據」。
**工程實踐**：使用明確的分隔符。
```markdown
### Instructions ###
Summarize the following text. Focus on technical bottlenecks.
### Input Data ###
[text]
```
這種做法能有效防止 **Prompt Injection (指令注入)**，並讓模型在處理長文本時能快速定位目標內容。
## 確定性調校：Temperature 與 Top-P
很多 AI 應用工程師在 Prompt 上花了一個月，卻忽略了 API 參數。
- **Temperature $\rightarrow$ 0**：這是工程師的預設值。如果你在做數據提取、代碼生成或 RAG，你需要的是 **決定論 (Determinism)**。溫控越低，模型越傾向於選擇概率最高的 token，輸出越穩定。
- **Top-P (Nucleus Sampling)**：如果你需要一點創意但又不希望模型胡言亂語，可以保持中高溫度，但將 Top-P 設低（例如 0.8）。這會強迫模型只在最可能的 80% Token 池中選擇，剔除掉那些極低概率的雜訊。
## 總結：從 Prompt Engineer 轉向 AI System Engineer
如果你還在試圖尋找那個「能讓模型變聰明」的神奇詞彙，你依然在煉金。
真正的 AI 應用工程，是將 Prompt 視為 **「接口定義」**。你的目標不是寫出優美的文字，而是建立一套**可預測、可模組化、可評測**的指令系統。
**記住：最好的 Prompt 不是讓模型「變聰明」，而是讓模型「沒機會犯錯」。**


---

## 🔗 Related Insights

- How prompt structure affects the limited context window: Context Window Dynamics [TBD]
- Why the best prompt can't fix a broken retrieval pipeline: [Rag Truth](rag-truth.md)