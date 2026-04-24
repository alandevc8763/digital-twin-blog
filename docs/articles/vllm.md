# 記憶體才是真正的瓶頸：vLLM 如何用「分頁」重構 LLM 推理

當大多數人還在爭論哪個模型更強時，生產環境的工程師在爭論一件更現實的事：**GPU 記憶體怎麼才夠用？**

在部署 LLM 時，最令人頭痛的不是計算量 (Compute)，而是 KV Cache (Key-Value Cache) 佔用的記憶體。傳統的推理引擎在處理長文本或高併發請求時，記憶體碎片化極其嚴重，導致 GPU 雖然還有空間，卻無法承載更多請求。

這就是為什麼 **vLLM** 的出現被視為推理引擎的分水嶺。它引入了一個堪比作業系統分頁機制 (Virtual Memory) 的創新：**PagedAttention**。

## PagedAttention：LLM 推理的「分頁管理」時刻

在 vLLM 出現之前，KV Cache 必須以連續的記憶體塊形式儲存。這就像在飯店訂房，如果你預訂了一個大套房但只住一個人，剩下的空間不能分給別人，造成巨大的浪費。

PagedAttention 改變了這個遊戲規則。它將 KV Cache 劃分為固定大小的「頁面 (Pages)」，並透過一個映射表 (Mapping Table) 進行管理。

這帶來了三個革命性的改變：
1. **零碎片化 (Zero Fragmentation)**：記憶體不再需要連續， wherever a free page exists, it can be used. 記憶體利用率接近 100%。
2. **高效共享 (Efficient Sharing)**：在處理相同前綴 (Prompt Prefix) 的多個請求時，多個請求可以共享同一個物理頁面。這讓並發處理能力呈數量級提升。
3. **動態擴展**：記憶體可以根據生成的 token 數量動態分配，不再需要預先估計最大長度。

## 除了分頁，vLLM 還有哪些「黑科技」？

PagedAttention 是核心，但 vLLM 的高性能來自於一系列對底層的極致榨取：

- **Continuous Batching (連續批處理)**：傳統 Batching 必須等整個批次全部生成完才能開始下一輪。vLLM 允許在任何一個請求完成後立即插入新請求，讓 GPU 永遠處於滿載狀態。
- **優化內核 (Optimized Kernels)**：深度整合了 FlashAttention 和 FlashInfer，將 Attention 的計算複雜度降到最低。
- **極強的適配力**：無論是 NVIDIA、AMD 還是 TPU，vLLM 幾乎支持所有主流硬體，並且對 FP8、AWQ 等量化格式有原生支持。

## 從「能跑」到「能規模化」

對開發者來說，vLLM 的價值在於它將 LLM 的部署從「實驗室產品」推向了「工業級產品」。

如果你需要：
- 在同一張 GPU 上承載 5 倍的併發量。
- 降低單個 token 的推理成本。
- 快速部署 Llama 3 或 DeepSeek 等開源模型並提供 OpenAI 相容 API。

那麼 vLLM 幾乎是目前的唯一選擇。

## 結語

LLM 的競爭最終會進入「效率之戰」。當模型能力趨同時，誰能用更少的資源提供更快的回應，誰就贏得了生產環境的門票。

vLLM 證明了：**最好的 AI 優化，往往不是在模型層面，而是在系統層面。**

---
**技術關鍵字：**
- PagedAttention (分頁注意力)
- Continuous Batching (連續批處理)
- KV Cache Optimization (KV 快取優化)
- High-Throughput Serving (高吞吐量服務)
