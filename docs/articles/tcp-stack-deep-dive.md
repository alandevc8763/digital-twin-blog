# 當你的服務出現 Packet Loss 時，底層到底發生了什麼？：TCP/IP 棧與 Conntrack 深度追蹤

對於大多數開發者來說，發送一個網路請求就像是按下開關 $ightarrow$ 等待燈亮。但在 SRE 的眼中，一個封包在 Kubernetes 叢集中的旅程，根本不是簡單的「發送」，而是一場複雜的**路徑導向 (Steering)**。

如果你曾經遇到過這種情況：`ping` 沒問題，小請求能通，但大數據量時連接突然「掛掉」 (Hanging)，或者在 CPU 沒滿的情況下出現隨機的 `Connection Timeout` —— 那麼你面對的可能不是應用程式 Bug，而是 Linux 核心網路棧的底層陷阱。

## 封包的「垂直旅程」：從 Pod 到物理網卡

在 K8s 中，一個封包從 Pod 出發到到達物理網卡，會經歷多次 **Context Handoffs (上下文交接)**。這正是潛在延遲和丟包的來源。

### 1. Pod 命名空間 (The Source)
應用程式呼叫 `connect()`，核心在 Pod 的網路命名空間 (`netns`) 建立 Socket。此時封包被綁定在 Pod 的本地路由表。
- **追蹤點**：`Pod App` $ightarrow$ `Socket Buffer (sk_buff)` $ightarrow$ `eth0`。

### 2. 命名空間跳躍 (The Jump)
封包離開 Pod 的 `eth0`，通過一個 **Virtual Ethernet (`veth`) pair**。`veth` 就像一條虛擬補丁線，將封包從 Pod 的命名空間直接「拉」到主機的 root 命名空間。
- **SRE 訊號**：如果這裡丟包，通常意味著 CNI 插件崩潰或介面被意外刪除。

### 3. 主機導向邏輯 (Netfilter & Kube-Proxy)
進入 root 命名空間後，封包會撞上 **Netfilter** 的鉤子。如果目標是 `ClusterIP`，`iptables` 或 `IPVS` 會執行 **DNAT (目的地址轉換)**，將虛擬 IP 改為真實的 Pod IP。
- **致命瓶頸：Conntrack (連接追蹤)**
  核心必須記錄每一次轉換的狀態。如果 `nf_conntrack_max` 達到上限，核心會**靜默丟棄 (Silent Drop)** 封包。這就是為什麼在高併發環境下，你會看到隨機的連線失敗，而日誌裡卻找不到任何錯誤。

### 4. TCP 狀態機 (The Engine)
最後，核心處理 TCP 協議邏輯：管理三次握手、計算擁塞窗口 (`cwnd`) 以及處理重傳定時器 (RTO)。

---

## 實戰案例：為什麼大請求會「掛掉」？ (MTU 黑洞)

這是 SRE 面試中最經典的場景：**小請求秒回，大請求超時。**

**現象**：API 呼叫 `GET /health` 正常，但 `GET /large-report` 導致連線永久掛起。

**底層 Trace 過程**：
1. **抓包分析**：在 Pod 和 Host 同時 `tcpdump`。發現伺服器發出了 1500 bytes 的大封包，但客戶端永遠沒有回傳 `ACK`。
2. **MTU 驗證**：檢查發現 Pod 的 MTU 是 1500，但 CNI 的 Overlay 網路 (如 VXLAN) 因為封裝開銷，實際 MTU 只有 1450。
3. **根因**：這是一個 **「MTU 黑洞」**。封包太大，中間路由器丟棄了它並發出 `Fragmentation Needed` 訊號，但該訊號被防火牆攔截了。發送端在傻傻地等待一個永遠不會到來的 `ACK`。

**解決方案**：統一對齊 MTU，或開啟 **TCP MSS Clamping**，強制在握手階段協商較小的最大段大小。

## 結語：從「看日誌」到「看軌跡」

真正的 SRE 不應該依賴 `kubectl logs`，而應該依賴於對 **軌跡 (Trace)** 的理解。

當你能從 `ss -ntie` 查看 `cwnd`，用 `conntrack -L` 檢查狀態表，用 `tcpdump` 對比 `veth` 與 `eth0` 的時間戳時，網路就不再是一個黑盒，而是一條清晰的數據流。

---
**技術關鍵字：**
- Netfilter / Conntrack (連線追蹤)
- veth Pair (虛擬乙太網)
- MTU Mismatch (最大傳輸單元不匹配)
- TCP State Machine (TCP 狀態機)
- DNAT / ClusterIP (目的地址轉換)
