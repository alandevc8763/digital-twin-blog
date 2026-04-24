---
title: "Claude Context"
date: 2026-04-24
categories: ["General"]
tags: ["General", "Claude-Context.Md"]
draft: false
---

# 當 Context Window 依然不夠用時：用 Claude Context 構建程式碼的「語義地圖」

很多開發者在面對 200K 甚至 1M 的上下文窗口 (Context Window) 時，容易陷入一個誤區：認為只要把所有程式碼全部塞進去，LLM 就能理解一切。

但事實是，即使窗口夠大，**「大海撈針 (Needle In A Haystack)」** 效應依然存在。過多的冗餘資訊會稀釋關鍵信號，導致模型產生幻覺或遺漏細節。對於真正的生產級 codebase 來說，我們需要的不是更大的窗口，而是一個能精確導航的 **「語義地圖」**。

這就是 **Claude Context** 試圖解決的問題。

## 為什麼簡單的 RAG 對於程式碼來說是失敗的？

大多數的 RAG (檢索增強生成) 系統在處理文本時表現良好，但在處理程式碼時往往表現糟糕。原因在於：
- **塊分割 (Chunking) 太隨機**：傳統 RAG 按照字符數分割，經常把一個函數切成兩半，導致語義丟失。
- **缺乏結構意識**：它不理解函數調用鏈、類繼承或模組依賴，只把程式碼當成普通的字符串。

Claude Context 透過以下技術打破了這個僵局：

### 1. AST-Aware Chunking (基於抽象語法樹的切分)
它不再隨機切分，而是利用 **AST (Abstract Syntax Tree)** 根據程式碼的邏輯結構（如函數邊界、類定義）進行切分。這確保了被檢索出來的每一個片段都是一個「完整的邏輯單元」。

### 2. 混合搜索 (Semantic Hybrid Search)
單純的向量搜索 (Dense Vector Search) 有時會太過「模糊」，而關鍵字搜索 (BM25) 又太過「死板」。Claude Context 將兩者結合：
- **精確匹配**：快速定位特定的函數名或變量名。
- **概念關聯**：即使你沒有使用精確的詞彙，也能通過語義關聯找到相關邏輯。

### 3. 增量索引 (Incremental Indexing)
對於大型項目，重新索引整個 codebase 是不可接受的。它引入了 **Merkle Tree** 機制，僅對修改過的檔案進行更新，確保索引始終與最新代碼同步，且幾乎不消耗額外資源。

## 從「搜索」到「Agentic Refactoring」

當 Claude Context 作為 MCP (Model Context Protocol) 插件集成到 AI Agent 中時，它賦予了 Agent 一種全新的能力：**深層探索 (Deep Exploration)**。

在進行大規模重構時，Agent 不再是盲目地修改一個檔案，而是：
`語義搜索相關函數` $
ightarrow$ `分析依賴關係` $
ightarrow$ `定位所有受影響的呼叫端` $
ightarrow$ `制定全局重構計劃` $
ightarrow$ `執行修改`。

這種能力將 AI 從一個「代碼片段生成器」提升到了「系統架構理解者」的高度。

## 結語：上下文的質量高於數量

我們正處在一個對抗「token 爆炸」的時代。真正的效率提升不在於增加窗口大小，而在於提高**上下文的信噪比 (Signal-to-Noise Ratio)**。

Claude Context 提供了一種典範：透過將 domain-specific 的結構知識（如 AST）與現代向量檢索結合，我們可以為 LLM 構建一個高效的外部記憶體。

---
**技術關鍵字：**
- MCP (Model Context Protocol)
- AST-Aware Chunking (基於 AST 的切分)
- Semantic Hybrid Search (語義混合搜索)
- Merkle Tree Indexing (默克爾樹索引)