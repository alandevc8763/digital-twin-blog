# 公平與延遲的戰爭：解構 Linux CFS 調度器
在 Linux 世界裡，CFS (Completely Fair Scheduler) 是絕對的主角。它的名字裡有一個詞非常關鍵：**Fair (公平)**。
但對於高性能應用工程師來說，「公平」往往是低延遲的最大敵人。
## vruntime：公平的代價
CFS 不使用傳統的「時間片」概念，而是使用一個紅黑樹來追蹤每個進程的 `vruntime` (虛擬運行時間)。
- 誰的 `vruntime` 最小 $
ightarrow$ 誰最「委屈」 $
ightarrow$ 誰優先被調度執行。
這種機制保證了系統不會有進程被完全餓死，但它也導致了一個問題：**調度抖動**。
當大量短暫的任務（例如大量的小請求）同時進入系統時，CFS 會為了維持「公平」，頻繁地在不同任務之間切換。這種 **Context Switch Storm (上下文切換風暴)** 會導致 CPU 緩存被不斷刷新，讓你的 P99 延遲劇烈飆升。
## 繞過「公平」：追求確定性執行
如果你在構建一個實時數據處理管線，你不需要公平，你需要的是 **確定性 (Determinism)**。
### 1. CPU 隔離 (isolcpus)
最極端的做法是在內核啟動參數中加入 `isolcpus=1,2,3`。這告訴內核：「不要把任何普通任務調度到這幾個核心上」。然後，你使用 `taskset` 將你的關鍵進程強行綁定到這些隔離核心上。
這樣，你的進程在該核心上擁有 **絕對的霸權**，不再受 CFS 的公平調度干擾。
### 2. 實時調度策略 (Real-Time Policy)
透過 `chrt` 將進程設置為 `SCHED_FIFO` 或 `SCHED_RR`。這讓你的進程跳出了 CFS 的紅黑樹，只要它想運行，就能立即搶佔任何普通進程。
## 總結：公平是政策，不是性能
CFS 是為了讓通用作業系統「好用」，但它不是為了讓高性能應用「快」。**真正的性能優化，就是學會如何在正確的地方放棄公平。**


---

## 🔗 Related Insights

- The physical topology that underlies the scheduler: [Cpu Numa](cpu-numa.md)
- Tuning the scheduler for high-throughput LLM inference: [Local Ai Infra](../ai-engineering/local-ai-infra.md)
