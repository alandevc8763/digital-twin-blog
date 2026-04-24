---
title: "K8S Networking Plumbing"
date: 2026-04-24
categories: ["General"]
tags: ["General", "K8S-Networking-Plumbing.Md"]
draft: false
---

# K8s 網路底層揭秘：從 Pod Sandbox 到 ClusterIP 的封包旅程

如果你問一個 K8s 開發者「Service 是什麼？」，他可能會告訴你「它是一個穩定的虛擬 IP，負責把流量分發到後端 Pod」。

但在 SRE 眼中，**Kubernetes 的 Service 是一個巨大的謊言。**

事實上，ClusterIP 並不是一個真實的網路設備，也沒有一個真正的伺服器在監聽那個 IP。它其實是一個**分佈式核心級防火牆規則**。當你發送一個請求到 Service IP 時，封包在到達目標之前，已經在 Linux 核心中被「悄悄地」修改了。

## 1. Pod 的「隔離單元」：Pod Sandbox 與 Pause 容器

在深入網路路徑前，得先理解 Pod 的物理結構。K8s 的 Pod 並不是一個單一的容器，而是一個 **Sandbox (沙箱)**。

### 錨點：Pause 容器
每當一個 Pod 啟動時，Kubelet 會先啟動一個幾乎不佔資源的 `pause` 容器。它的唯一作用就是**持有**一組命名空間 (Namespaces)，包括：
- **Network Namespace (NET)**：一套私有的 IP、路由表與端口。
- **IPC Namespace**：私有的共享記憶體。

隨後啟動的所有應用容器，都會通過 `setns()` 系統呼叫，「寄生」在 `pause` 容器的命名空間中。這就是為什麼同一個 Pod 內的兩個容器可以用 `localhost` 互相通信，且不能綁定同一個端口 $	ext{(:8080)}$ 的原因 $
ightarrow$ 因為它們共享同一個核心網路對象。

## 2. 封包的垂直旅程：從 Ingress 到 Pod IP

當一個外部請求進入叢集，它經歷的路徑如下：

### 第一階段：L7 導航 (Ingress $
ightarrow$ Service)
Ingress Controller (如 Nginx) 接收請求 $
ightarrow$ 根據 Host Header 找到對應的 `Service` $
ightarrow$ 查表獲取後端 `Endpoints` (即真實的 Pod IP 列表)。

### 第二階段：核心變異 (ClusterIP $
ightarrow$ Pod IP)
如果請求是通過 ClusterIP 發送的，封包會撞上 Linux 核心的 **Netfilter/iptables** 鏈：
1. **攔截**：封包進入 `PREROUTING` 鏈。
2. **隨機選擇**：`KUBE-SVC-XXX` 鏈隨機選擇一個後端 Pod IP。
3. **DNAT (目的地址轉換)**：核心直接修改 IP 標頭，將目的地址從 `ClusterIP` 替換為 `PodIP`。

**這就是關鍵：** 封包在離開主機網卡前，目的地址就已經變了。

### 第三階段：最後一公里 (Veth Pair $
ightarrow$ Pod)
封包通過 **Veth Pair (虛擬乙太網對)**，從主機的 root 命名空間「跳」進 Pod 的網路命名空間 $
ightarrow$ 到達 Pod 的 `eth0` $
ightarrow$ 交付給應用程式。

---

## 3. SRE 除錯矩陣：當網路斷掉時，看哪裡？

面對網路故障，最快的方法是**儘快拋棄 Service 這個抽象概念，直接對接 Pod IP。**

| 故障現象 | 診斷路徑 | 核心原語 | 推薦工具 | 結論 |
| :--- | :--- | :--- | :--- | :--- |
| **`no healthy upstream`** | Service $
ightarrow$ Endpoints | Endpoint List | `kubectl get ep` | 應用程式崩潰或 Readines Probe 失敗 |
| **`Connection Refused`** | Node $
ightarrow$ Pod | veth / Routing | `curl <pod-ip>` | 應用程式未監聽端口或 CNI 路由失效 |
| **`Connection Timeout`** | Netfilter $
ightarrow$ DNAT | `nf_conntrack` | `tcpdump` / `iptables-save` | Conntrack 表滿導致靜默丟包 |

## 4. 避坑指南：那些隱形的網路陷阱

- **Hairpinning (髮夾彎)**：如果 Pod 嘗試呼叫自己的 Service ClusterIP，流量會繞一圈回來。如果 CNI 不支援 Hairpin 模式，這個請求會直接失敗。
- **Conntrack 飽和**：在高併發環境下，核心的連線追蹤表 (`nf_conntrack`) 會被填滿。這會導致隨機的 `Connection Timeout`，即便 Pod 狀態是 `Running` 且 CPU/Mem 充足。
- **DNS 掩蓋**：很多 `502` 錯誤其實是 CoreDNS 延遲或失敗導致 Ingress 無法解析 Service 名稱，而非後端服務的問題。

## 結語：回歸底層

K8s 網路的複雜之處在於它用大量的「虛擬化」掩蓋了底層的簡單。

當你在除錯時，請記得這個順序：**Pod IP $
ightarrow$ Veth Pair $
ightarrow$ iptables/IPVS $
ightarrow$ Ingress**。只要你能證明 `curl <pod-ip>` 是通的，你就已經解決了 80% 的網路問題。

---
**技術關鍵字：**
- Pod Sandbox (Pod 沙箱)
- Pause Container (錨點容器)
- DNAT (目的地址轉換)
- Veth Pair (虛擬乙太網對)
- nf_conntrack (連線追蹤)
- ClusterIP (虛擬 IP)