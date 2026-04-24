---
title: "K8S Resource Limits"
date: 2026-04-24
categories: ["General"]
tags: ["General", "K8S-Resource-Limits.Md"]
draft: false
---

# K8s Resource Limits 的真相：為什麼你的 CPU 使用率很低，延遲卻在飆升？

在 Kubernetes 的 YAML 設定中，`resources.limits` 是每個開發者都會寫的配置。但大多數人對它的理解僅停留在「限制資源使用量」這個表面層級。

事實上，K8s 的 Limits 不是「預留 (Reservation)」，而是「天花板 (Ceiling)」。而且，CPU Limit 和 Memory Limit 在核心層級的執行機制截然不同：**一個導致的是「慢 (Throttling)」，另一個導致的是「死 (OOM Kill)」。**

## 1. CPU Limit：隱形的「凍結」陷阱

許多 SRE 曾遇到過這種詭異現象：`kubectl top pod` 顯示 CPU 使用率僅 30%，但服務的 P99 延遲卻從 20ms 飆升到 200ms。

### 核心機制：CFS Quota (完全公平調度配額)
當你設定 `limits.cpu: 500m` 時，核心並不是在監控每秒的平均值，而是在執行一個**時間窗 (Window)** 遊戲：
- **周期 (Period)**：預設為 100ms。
- **配額 (Quota)**：50ms。

如果你的一個請求觸發了 4 個執行緒並行處理，每個執行緒消耗 25ms，那麼在短短 **25ms 的牆鐘時間 (Wall-clock time)** 內，你就用完了整個 100ms 周期所有的 50ms 配額。

**結果**：核心會直接將你的整個 Cgroup **暫停 (Deschedule)**，強迫你進入 75ms 的「冷凍期」。這就是為什麼你的平均 CPU 使用率看起來很低，但每個請求卻被強行注入了 75ms 的延遲。

### SRE 如何捕捉這個訊號？
不要相信 `kubectl top`，直接去看核心的統計：
`cat /sys/fs/cgroup/cpu.stat` $\rightarrow$ 觀察 `nr_throttled` 是否在快速增加。如果這個數字在跳動，說明你的服務正在被核心「掐脖子」。

---

## 2. Memory Limit：殘酷的「死刑」

與 CPU 不同，記憶體是一種**不可壓縮資源**。你不能透過「讓記憶體慢一點」來解決壓力。

### 核心機制：Hard Limit 與 OOM Killer
當容器的記憶體使用量 (RSS + Cache) 觸及 `memory.max` 時，核心會嘗試進行頁快取回收 (Page Cache Reclaim)。如果回收後依然無法滿足新的分配請求，核心會啟動 **OOM Killer**。

**執行路徑**：
`記憶體壓力` $\rightarrow$ `嘗試回收快取` $\rightarrow$ `回收失敗` $\rightarrow$ `計算 oom_score` $\rightarrow$ `發送 SIGKILL`。

**SRE 訊號**：
- `kubectl describe pod` $\rightarrow$ `Reason: OOMKilled`。
- `dmesg -T | grep -i oom` $\rightarrow$ 看到明確的 `Memory cgroup out of memory: Killed process...`。

---

## 3. 資源調優矩陣：從現象到根因

| 觀察現象 | 檢查維度 | 核心訊號 | 診斷結論 | 建議動作 |
| :--- | :--- | :--- | :--- | :--- |
| **P99 延遲飆升 / CPU 貌似不高** | Cgroup CPU Stat | `nr_throttled` $\uparrow$ | **CPU Throttling** | 提高 `limits.cpu` 或優化並行度 |
| **Pod 隨機重啟 / 狀態 OOMKilled** | Kernel Log | `oom_score` $\uparrow$ | **Memory Limit Breach** | 檢查記憶體洩漏或提高 `limits.memory` |
| **啟動緩慢 / 響應遲鈍** | CPU Weight | `cpu.weight` 低且節點擁擠 | **CPU Contention** | 提高 `requests.cpu` 以獲取更高權重 |

## 結語：不要被 YAML 欺騙

`resources.limits` 是一把雙面刃。設定太高會導致節點資源碎片化，設定太低則會讓你的服務在核心層級遭遇「靜默處決」。

真正的資源調優，應該是**基於 `cpu.stat` 的 Throttling 數據**與 **`dmesg` 的 OOM 記錄**，而不是拍腦袋決定一個數字。

---
**技術關鍵字：**
- CFS (Completely Fair Scheduler)
- CPU Throttling (CPU 節流)
- OOM Killer (記憶體溢出殺手)
- Cgroup v2 (控制組)
- BDP (Bandwidth-Delay Product)