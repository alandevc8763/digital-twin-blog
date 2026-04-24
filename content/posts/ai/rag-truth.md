---
title: "Rag Truth"
date: 2026-04-24
categories: ["AI"]
tags: ["AI", "Ai-Engineering", "Rag-Truth.Md"]
draft: false
---

# RAG 的真相：為什麼你的向量搜索總是找不準？
如果你剛開始構建 RAG (檢索增強生成) 系統，你的路徑大概率是這樣的：將文檔切成 500 個 token 的塊 $\rightarrow$ 調用 Embedding 接口 $\rightarrow$ 存入向量數據庫 $\rightarrow$ 根據相似度取 Top-K $\rightarrow$ 餵給 LLM。
在測試集上，這看起來很完美。但一旦進入生產環境，你會發現 LLM 開始頻繁出現這種狀況：
「雖然文檔裡明明有答案，但模型卻告訴我找不到」，或者「檢索回來的內容雖然看起來相關，但對回答問題完全沒幫助」。
這就是 **Naive RAG (天真 RAG)** 的陷阱。
## 向量相似度 $\neq$ 答案相關度
很多開發者對向量搜索有一個致命的誤解：認為「語義相似」就等於「能回答問題」。
**事實上，向量空間的距離（Cosine Similarity）捕捉的是「主題的接近程度」，而非「邏輯的相關性」。**
舉個例子：如果你搜索「如何修復 K8s 的 CrashLoopBackOff」，向量搜索可能會給你返回一篇關於「K8s 故障排除指南」的長文，因為兩者的語義高度相關。但你真正需要的是一段具體的 `kubectl` 指令或一個特定的配置文件範例。
這就是為什麼你的系統會出現「檢索到了，但沒用」的情況。
## 第一道防線：別讓分塊 (Chunking) 毀了你的數據
很多人把分塊當成一個簡單的 `split()` 操作，設定個 500 token，加個 50 token 的重疊（Overlap）就完事了。
這是最糟糕的做法。**固定大小的分塊會暴力地切斷邏輯鏈條。**
想像一下，一個關鍵的答案正好被切在第 500 個 token 的位置，前半段在塊 A，後半段在塊 B。當向量搜索只取 Top-1 時，LLM 拿到的只有半個答案。
**工程實踐的演進路徑：**
- **遞迴分塊 (Recursive Character Chunking)**：優先按段落切 $\rightarrow$ 再按句子切 $\rightarrow$ 最後按單詞切。這能最大限度保留結構完整性。
- **語義分塊 (Semantic Chunking)**：不再設定固定長度，而是利用 Embedding 計算相鄰句子之間的餘弦相似度，當相似度掉到臨界值以下時，才判定為「話題切換」，在此處切分。
- **結果**：你的 Chunk 變成了一個個獨立的「知識單元」，而非被切碎的文字碎片。
## 第二道防線：混合搜索 (Hybrid Search) 才是正解
單純依賴 Dense Embedding (向量搜索) 會讓你喪失對 **「精確匹配」** 的控制力。
如果你搜索一個特定的錯誤代碼 `0x8004210B`，向量搜索可能會給你返回一堆關於「郵件發送失敗」的文檔（因為語義相關），但它可能找不到那篇唯一提到這個錯誤碼的修復指南。
**SRE 級別的解決方案是 Hybrid Search：**
$\text{最終得分} = \alpha \cdot \text{向量分數 (Semantic)} + (1-\alpha) \cdot \text{BM25 分數 (Keyword)}$
透過結合 **BM25 (關鍵字搜索)**，你保證了「精確名詞」能被命中；透過 **向量搜索**，你保證了「意圖」能被捕捉。
## 最後的殺手鐧：精排 (Re-ranking)
即便有了 Hybrid Search，Top-K 依然是一個粗糙的過濾器。
**真正的 RAG 高手會在 LLM 之前加上一個 Re-ranker (精排模型)。**
- **粗排 (Retrieval)**：從 100 萬個塊中快速找出 50 個「大概相關」的塊（速度快，精度中）。
- **精排 (Re-ranking)**：使用 Cross-Encoder 模型對這 50 個塊進行兩兩比對，重新打分，選出最強的 5 個（速度慢，精度極高）。
Re-ranking 的 ROI 是 RAG 流程中最高的。它能把那些「看起來很像但沒用」的噪聲剔除，確保餵給 LLM 的 Context 是純淨且高相關的。
## 總結：RAG 其實是數據工程問題
請停止在 Prompt 上糾結如何讓模型「更好地閱讀 Context」。
如果檢索回來的內容是垃圾，再強的 LLM 也變不出金子。**RAG 的核心競爭力不在於你用了哪個 LLM，而是在於你如何處理數據分塊、如何設計混合搜索以及如何執行精排。**
把 RAG 當成一個 **數據清洗與過濾管線** 來設計，而不是當成一個聊天機器人來調教。


---

## 🔗 Related Insights

- The interaction between retrieval chunks and the context window: Context Window Dynamics [TBD]
- Running a high-performance RAG stack on local hardware: [Local Ai Infra](local-ai-infra.md)