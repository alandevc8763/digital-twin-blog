---
title: "Ebpf Modern Observability"
date: 2026-04-24
categories: ["General"]
tags: ["General", "Ebpf-Modern-Observability.Md"]
draft: false
---

# 超越 tcpdump：用 eBPF 將 Linux 核心轉化為可編程的監控引擎

對於大多數工程師來說，診斷網路或系統問題的工具箱通常是：`ping` $
ightarrow$ `curl` $
ightarrow$ `tcpdump` $
ightarrow$ `strace`。但在面對極端的高併發環境或「幽靈般」的延遲尖峰時，這些工具會顯得力不從心。

`tcpdump` 只能告訴你封包是否到達了網卡，但不能告訴你封包在核心內部為什麼被丟棄；`strace` 能看到系統呼叫，但會讓進程速度大幅下降，甚至改變 Bug 的觸發條件（即 Heisenbug）。

這正是 **eBPF (Extended Berkeley Packet Filter)** 登場的意義。它將 Linux 核心從一個靜態的二進制文件，變成了一個**可編程的平台**。

## eBPF 的本質：在核心內部植入「探針」

簡單來說，eBPF 允許 SRE 在不需要重新編譯核心、不需要重啟伺服器且不風險導致核心崩潰 (Kernel Panic) 的情況下，將自定義邏輯直接注入到核心的執行路徑中。

### 1. 從代碼到執行的「垂直路徑」
一個 eBPF 程序的旅程如下：
**SRE 腳本 (`bpftrace`/`Cilium`)** $
ightarrow$ **LLVM 編譯** $
ightarrow$ **`bpf()` 系統呼叫** $
ightarrow$ **Verifier (驗證器)** $
ightarrow$ **JIT 編譯** $
ightarrow$ **核心掛鉤點 (Hook Point)** $
ightarrow$ **事件觸發**。

其中最關鍵的是 **Verifier (驗證器)**。它會對字節碼進行靜態分析，確保程序沒有無限循環、沒有越界訪問內存。這就是為什麼 eBPF 能在核心中運行而不會導致系統崩潰的原因。

### 2. 選擇你的「掛鉤點」 (Hook Points)

根據你要解決的問題，你可以選擇不同的掛鉤點：

| 掛鉤類型 | 層級 | SRE 訊號 / 應用場景 | 推薦工具 |
| :--- | :--- | :--- | :--- |
| **kprobes** | 核心函數 | 追蹤核心內部函數的延遲、狀態偏移 | `bpftrace`, `bcc` |
| **uprobes** | 用戶空間 | 追蹤應用層延遲 (例如：SSL 握手耗時) | `bpftrace`, `bcc` |
| **tracepoints** | 靜態標記 | 觀察穩定的系統事件 (例如：調度器切換) | `perf`, `ftrace` |
| **XDP** | 網卡驅動 | 處理入站封包丟棄、DDoS 防護、L3/L4 過濾 | `Cilium`, `Hubble` |
| **LSM** | 安全模組 | 攔截未授權的系統呼叫、特權提升嘗試 | `Tetragon` |

---

## 實戰案例：追蹤「消失的封包」

這是 SRE 最頭痛的場景：**封包到了，但應用程式沒收到。**

**現象**：服務出現間歇性連線超時。`tcpdump` 顯示封包確實到達了物理網卡 $	ext{eth0}$，但應用程式日誌裡一片空白，完全沒有收到請求。

**診斷路徑**：
1. **確認網卡接收**：`tcpdump` 確認封包已到達。
2. **檢查棧丟包**：`netstat -s` 看到 `TCPBacklogDrop` 在增加，但它沒告訴我們封包在哪裡被丟掉的。
3. **部署 eBPF 探針**：使用 `bpftrace` 直接掛鉤到核心的 `tcp_drop` 函數：
   ```bpftrace
   kprobe:tcp_drop {
     printf("發現丟包! SKB: %p, 原因: %s\n", args->skb, kstack);
   }
   ```
4. **分析結果**：Trace 顯示 `tcp_drop` 在 `tcp_v4_rcv` 中被觸發，原因是 Socket 的接收隊列已滿 (`sk_max_buff` 超限)。
5. **根因**：應用程式處理封包的速度慢於到達速度，導致核心在封包進入 Socket 緩衝區之前就將其靜默丟棄。

**解決方案**：調優 `net.core.rmem_max` 並優化應用程式的 Event Loop 併發能力。

## 結語：從「猜測」到「確定」

當標準工具失效時，eBPF 提供了那塊缺失的拼圖。它讓你能夠觀察**硬件與應用程式之間的過渡地帶**。

如果你發現某個資源在核心內部「消失」了，或者某個系統呼叫莫名其妙地變慢，不要再嘗試用 `strace` 這種會干擾性能的工具。直接在相關的 `kprobe` 或 `tracepoint` 上部署 eBPF 探針，將「猜測」轉化為「確定的數據」。

---
**技術關鍵字：**
- eBPF (Extended Berkeley Packet Filter)
- Verifier (驗證器)
- JIT (Just-In-Time Compilation)
- kprobes / uprobes (核心/用戶探針)
- XDP (eXpress Data Path)
- Cilium / Tetragon (基於 eBPF 的雲原生工具)