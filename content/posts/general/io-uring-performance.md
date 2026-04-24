---
title: "Io Uring Performance"
date: 2026-04-24
categories: ["General"]
tags: ["General", "Io-Uring-Performance.Md"]
draft: false
---

# 告別系統呼叫稅：為什麼 io_uring 是高性能 Linux 服務的未來？

在高性能伺服器的開發中，我們一直面臨一個巨大的瓶頸：**系統呼叫 (Syscall) 稅**。

每當應用程式想要讀取檔案或發送網路封包時，它必須通過 `read()` 或 `write()` 進入核心。這次切換（Context Switch）伴隨著權限檢查、暫存器保存和快取失效。在每秒處理數百萬次 I/O 的極端場景下，這些微小的開銷會累積成巨大的效能損失。

`io_uring` 的出現，徹底改變了 Linux 處理 I/O 的方式。它不再是「請求 $
ightarrow$ 等待 $
ightarrow$ 回應」，而是一場基於 **共享記憶體環形緩衝區 (Ring Buffers)** 的非同步協作。

## 核心機制：兩把環形緩衝區

`io_uring` 的精髓在於它在用戶空間與核心空間之間建立了兩塊共享記憶體區域，徹底消除了大多數 I/O 操作中的系統呼叫需求。

### 1. 提交隊列 (Submission Queue, SQ)
應用程式不再呼叫 `read()`，而是將一個「提交條目 (SQE)」直接寫入 SQ 環形緩衝區。這個條目詳細描述了操作內容（例如：從哪個 FD 讀取多少資料到哪個緩衝區）。

### 2. 完成隊列 (Completion Queue, CQ)
核心處理完 I/O 後，會將結果（例如：讀到了多少個 Byte）寫入 CQ 環形緩衝區。應用程式只需要定期檢查 CQ 環，就能知道哪些操作已完成。

### 3. 從 $	ext{O(N)}$ 到 $	ext{O(1)}$ 的進化
- **傳統 I/O**：$N$ 次 I/O 操作 $
ightarrow$ $N$ 次系統呼叫。
- **io_uring**：$N$ 次 I/O 操作 $
ightarrow$ 1 次 `io_uring_enter()`（甚至 0 次，如果開啟了 `SQPOLL` 模式）。

---

## 垂直軌跡：一個 I/O 請求的生命週期

1. **提交 (Submission)**：應用程式在 SQ 中填寫一個 SQE $
ightarrow$ 呼叫 `io_uring_enter()` 通知核心（或由核心執行緒自動輪詢）。
2. **非同步執行 (Async Execution)**：核心接收到 SQE 後，將工作分派給內部工作執行緒或硬體驅動，立即將控制權交還給用戶空間。
3. **完成 (Completion)**：當硬體完成 I/O 後，核心將結果寫入 CQ 環。
4. **收割 (Harvesting)**：應用程式從 CQ 環中讀取結果，全程無需再次進入核心。

## SRE 的觀測視角：瓶頸在哪裡？

雖然 `io_uring` 極其強大，但它也引入了新的調優維度：

- **SQ 溢出 (SQ Overflow)**：如果提交速度快於核心處理速度，SQ 會滿，導致 `io_uring_enter` 阻塞。需要檢查 `sq_entries` 配置。
- **CQ 溢出 (CQ Overflow)**：如果應用程式讀取結果太慢，CQ 會滿，核心可能會停止處理新的請求。
- **SQPOLL 的代價**：開啟 `IORING_SETUP_SQPOLL` 可以實現零系統呼叫，但代價是核心會永遠佔用一個 CPU 核心來輪詢 SQ。這是在「CPU 消耗」與「極低延遲」之間的權衡。

## 結語：從同步到真正的非同步

`io_uring` 不僅僅是一個新 API，它代表了 Linux I/O 模型的一次範式轉移。它將 I/O 從「指令式」變成了「宣告式」：**我告訴核心我想做什麼，核心在完成後通知我。**

對於追求極致吞吐量的資料庫、快取系統或高併發 Gateway 來說，`io_uring` 是目前 Linux 平台上最強大的武器。

---
**技術關鍵字：**
- io_uring (非同步 I/O)
- Ring Buffer (環形緩衝區)
- SQE / CQE (提交/完成條目)
- SQPOLL (提交輪詢)
- Syscall Tax (系統呼叫開銷)