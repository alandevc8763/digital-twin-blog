---
title: "Network Tuning Deep Dive"
date: 2026-04-24
categories: ["General"]
tags: ["General", "Network-Tuning-Deep-Dive.Md"]
draft: false
---

# 從「黑洞」到 Bufferbloat：高併發環境下的 Linux 核心網路調優實戰

很多 SRE 在面對網路問題時，習慣於在 L7（應用層）找 Bug，或者在 L3（路由層）檢查連通性。但最棘手的問題往往發生在 L4（傳輸層）與核心之間的**灰色地帶**。

這裡有兩道隱形的牆：一道是 **Conntrack (連線追蹤)**，一旦滿了，你的服務會瞬間變成一個「黑洞」；另一道是 **Socket Buffers (套接字緩衝區)**，配置不當會導致你的延遲在沒有丟包的情況下莫名飆升。

## 第一道牆：Conntrack —— 靜默丟包的罪魁禍首

在 K8s 叢集中，`kube-proxy` 使用的 `iptables` 模式極度依賴核心的 `nf_conntrack` 模組。它為每個流 (Flow) 建立一個「影子狀態」，記錄封包的來源、目的地與狀態。

### 1. 當核心變成「黑洞」
Conntrack 的失敗模式是二元的：一旦達到 `nf_conntrack_max` 的上限，核心會直接**停止路由**，將新封包靜默丟棄 (Silent Drop)。

這最讓人崩潰的地方在於：**應用程式完全健康，CPU 沒滿，記憶體充足，但連線卻隨機超時。**

### 2. SRE 的除錯軌跡：追蹤「黑洞」連線
**現象**：高流量 API Gateway 開始出現 1% 的隨機 `Connection Timeout`。

**Trace 過程**：
1. **檢查核心日誌**：`dmesg -T | grep -i conntrack` $
ightarrow$ 發現 `nf_conntrack: table full, dropping packet`。
2. **量化飽和度**：`sysctl net.netfilter.nf_conntrack_count net.netfilter.nf_conntrack_max` $
ightarrow$ 發現 count 剛好等於 max。
3. **分析 churn (流轉率)**：使用 `conntrack -L` 分析發現，大量連線處於 `TIME_WAIT` 狀態。
4. **根因**：後端 legacy API 不支援 Keep-Alive，導致每次請求都建立新連線，填滿表單的速度快於核心回收的速度。

**對策**：
- **短期**：調高 `net.netfilter.nf_conntrack_max`。
- **長期**：在 Gateway 實作 Connection Pooling，或強制後端開啟 Keep-Alive。

---

## 第二道牆：Socket Buffers —— 吞吐量與延遲的拉鋸戰

如果說 Conntrack 是「門禁」，那麼 Socket Buffer (`sk_buff`) 就是核心的「緩衝墊」，用來解耦異步的網路傳輸與同步的應用程式 `read()/write()`。

### 1. Bufferbloat：沒有丟包的延遲陷阱
很多人認為「增加緩衝區 = 提高穩定性」。但在高頻交易或低延遲服務中，這是一個致命誤區。

**Bufferbloat (緩衝區膨脹)** 發生在緩衝區過大時。封包在長隊列中排隊，導致 RTT (往返時間) 劇增，但 TCP 擁塞控制算法因為沒看到丟包，以為網路依然暢通，於是繼續維持大窗口。結果就是：**吞吐量沒增加，延遲卻飆升。**

### 2. BDP 與吞吐量崩潰
相反，如果緩衝區設得太小，會觸發 **BDP (Bandwidth-Delay Product) Undershoot**。
$$	ext{BDP} = 	ext{頻寬} 	imes 	ext{往返延遲 (RTT)}$$
如果 `tcp_rmem` 或 `tcp_wmem` 低於 BDP，TCP 窗口無法開到足夠大，發送端在鏈路還有剩餘容量時就得停下來等 ACK，導致吞吐量遇到天花板。

### 3. SRE 訊號對照表

| 工具/指標 | 訊號 | 解讀 | 動作 |
| :--- | :--- | :--- | :--- |
| `ss -tmn` | 高 `recv-q` | **慢消費者**：應用程式 `read()` 太慢。 | 優化 Event Loop / 增加 Worker 執行緒。 |
| `ss -tmn` | 高 `send-q` | **慢網路/對端**：對端沒回 ACK 或窗口滿了。 | 檢查對端健康度 / 檢查路徑擁塞。 |
| `ping` | 高 RTT / 無丟包 | **Bufferbloat**：封包在深緩衝區排隊。 | 開啟 BBR 擁塞控制 / 縮小緩衝區。 |
| `sysctl` | `rmem_max` < BDP | **窗口限制**：吞吐量被緩衝區天花板封死。 | 調高 `net.core.rmem_max` 與 `tcp_rmem`。 |

## 結語：從「看指標」到「看機制」

網路調優不是在 `sysctl.conf` 裡隨機嘗試數字，而是在 **量化 BDP** 與 **追蹤 Conntrack 狀態** 之間尋找平衡。

當你發現服務延遲增加時，不要只看 CPU 負載，試著執行 `ss -tmn` 看看 `recv-q` 是否在堆積。如果你看到 `nf_conntrack: table full`，請記得：你的核心已經不再是路由器，而是一道牆。

---
**技術關鍵字：**
- nf_conntrack (連線追蹤)
- Bufferbloat (緩衝區膨脹)
- BDP (Bandwidth-Delay Product)
- TCP Window Scaling (TCP 窗口縮放)
- BBR Congestion Control (BBR 擁塞控制)