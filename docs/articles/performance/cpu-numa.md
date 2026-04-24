# 記憶體的物理學：CPU 拓撲與 NUMA 懲罰
很多人在調優高性能應用時，習慣於監控 CPU 使用率。當看到 CPU 只有 50% 但吞吐量卻上不去時，第一反應通常是「程式有鎖」或「網路慢」。
但對於跑在多路伺服器上的應用，真正的兇手往往是 **NUMA (Non-Uniform Memory Access)**。
## 記憶體訪問不再是「均一」的
在單路 CPU 時代，CPU 訪問記憶體就像在同一個房間拿東西。但在現代多路伺服器中，每個 CPU 都有自己的本地記憶體控制器。
如果你在 Socket 0 上運行一個線程，但它訪問的數據卻落在 Socket 1 的記憶體中，這個請求必須經過 **UPI (Intel)** 或 **Infinity Fabric (AMD)** 總線。這就是所謂的 **Remote Memory Access**。
**這導致了一個殘酷的事實：訪問遠端記憶體的延遲，比本地記憶體高出 30% 到 50%。**
## SRE 的實戰對策：追求極致的局部性 (Locality)
如果你在處理高頻交易、實時音頻或大規模資料庫，你不能把記憶體訪問交給內核隨機分配。
### 1. CPU Pinning 與記憶體親和性
不要讓內核在不同的核心之間隨機搬移你的線程（Context Switch）。使用 `numactl` 強制將進程綁定在特定的 NUMA 節點上：
`numactl --cpunodebind=0 --membind=0 ./my_app`
這確保了 **「計算」與「數據」在物理空間上是最近的**。
### 2. 禁用 SMT (超線程)
對於極致低延遲的場景，超線程 (Hyper-Threading) 其實是一種干擾。兩個邏輯核心共享同一個物理算術邏輯單元 (ALU)，這會引入不可預測的抖動 (Jitter)。在追求 P99.99 穩定性的系統中，直接在 BIOS 裡關掉 SMT。
## 總結：性能就是物理距離
在底層工程中，性能的本質就是 **「減少數據移動的距離」**。無論是 L1 Cache $
ightarrow$ L3 Cache，還是 Local RAM $
ightarrow$ Remote RAM，每一微米的移動都代表著延遲。**如果你不管理拓撲，你就是在交「距離稅」。**


---

## 🔗 Related Insights

- Understanding how the Linux Scheduler manages these cores: [Cfs Scheduler](cfs-scheduler.md)
- Applying NUMA awareness to Local AI GPU passthrough: Local Ai Infra Proxmox [TBD]
