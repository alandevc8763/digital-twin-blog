# 容器不是虛擬機：深入理解 Linux Namespaces 與 Cgroups 的「隔離幻象」

在很多人的認知中，Docker 或 Kubernetes 的容器像是一個「輕量級的虛擬機」。但從 SRE 的底層視角來看，這是一個巨大的誤解。

容器根本不是一個獨立的對象，而是一個**被限制了視角 (Namespaces)** 且 **被管控了資源 (Cgroups)** 的普通 Linux 進程。

## 第一部分：Namespaces —— 決定你能「看到」什麼

如果說核心是一個巨大的開放空間，那麼 **Namespaces (命名空間)** 就是在空間中隔出的「透明隔間」。它不限制你的能力，但它改變了你的**視角**。

### 1. 視角的隔離 (The Illusion of Privacy)
當 `runc` 啟動容器時，它會通過 `clone()` 或 `unshare()` 系統呼叫，為進程創建多個獨立的命名空間：

- **Network Namespace (NET)**：你看到的是一套私有的網卡、路由表和防火牆規則。這就是為什麼你在容器內執行 `ip addr` 看到的 IP 與主機完全不同。
- **PID Namespace**：你看到的進程 ID 被重新編號。容器內的啟動進程永遠是 `PID 1`，但從主機視角看，它可能是一個隨機的 `PID 12345`。
- **Mount Namespace (MNT)**：你擁有一套私有的掛載表。你在容器內 `mount` 一個磁碟，主機完全感覺不到。
- **User Namespace (USER)**：最神奇的隔離。你可以是容器內的 `root`，但在主機眼中，你只是一個毫無權限的普通用戶。

### 2. SRE 除錯場景：追蹤「幽靈進程」
**現象**：主機 `top` 顯示某個進程 CPU 100%，但你 `kubectl exec` 進去後，容器內的 `top` 卻顯示一切正常。

**真相**：這就是 PID Namespace 的視角差。該進程可能在一個私有的命名空間中運行，或者它是一個被孤立的子進程。
**對策**：使用 `nsenter` 命令。它可以讓你「跳出」容器的視角，直接進入該進程的命名空間：
`nsenter -t <PID> -m -u -i -n -p top` $ightarrow$ 現在你看到的 `top` 才是真實的。

---

## 第二部分：Cgroups —— 決定你能「用多少」

如果 Namespaces 是「牆」，那麼 **Cgroups (Control Groups)** 就是「計量表」。它不關心你看到了什麼，它只關心你消耗了多少資源。

### 1. 資源管控的演進 (v1 $ightarrow$ v2)
早期的 Cgroups v1 非常混亂，CPU、記憶體、PID 各自有一套樹狀結構。而 **Cgroups v2** 引入了「統一層級 (Unified Hierarchy)」，讓 SRE 能將一個容器視為一個完整的資源實體。

### 2. 三大資源陷阱
作為 SRE，你最需要關注的是這三種「被限制」的訊號：

- **CPU Throttling (硬限額)**：
  當你設定 `limits.cpu: 500m` 時，核心會啟動 CFS (Completely Fair Scheduler) 監控。一旦你在一個週期內用完了 50ms 的配額，核心會**直接暫停 (Suspend)** 你的進程，直到下一個週期開始。
  **訊號**：CPU 使用率不高，但 p99 延遲劇烈飆升。檢查 `/sys/fs/cgroup/.../cpu.stat` 中的 `nr_throttled`。

- **OOM Killer (記憶體之錘)**：
  當 `memory.max` 被觸發，核心不再猶豫，直接發送 `SIGKILL`。
  **訊號**：Pod 突然重啟，`kubectl describe` 顯示 `OOMKilled`。

- **PID Exhaustion (進程枯竭)**：
  為了防止 Fork-Bomb，核心會限制一個 Cgroup 能創建的進程總數 (`pids.max`)。
  **訊號**：Java 應用報 ` unable to create new native thread`。這通常不是記憶體不足，而是觸發了 PID 限制。

## 結語：從「黑盒」到「透明」

理解 Namespaces 與 Cgroups，是從「會用 K8s」晉升到「能調優 K8s」的分水嶺。

下次當你的 Pod 出現奇怪的網路問題或延遲時，請記得：**不要只看 K8s 的 YAML，要去 `/proc` 和 `/sys/fs/cgroup` 裡找真相。** 容器只是個幻象，底層始終是 Linux 核心在掌控一切。

---
**技術關鍵字：**
- Linux Namespaces (命名空間)
- Cgroups v2 (控制組)
- CFS Throttling (CPU 限額)
- OOM Killer (記憶體回收)
- nsenter (命名空間切換)
- PID 1 (初始化進程)
