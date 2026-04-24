---
title: "K8S Pod Lifecycle"
date: 2026-04-24
categories: ["General"]
tags: ["General", "K8S-Pod-Lifecycle.Md"]
draft: false
---

# K8s Pod 生命週期：為什麼 API 狀態總是在「欺騙」SRE？

對於大多數開發者來說，檢查 Pod 狀態的習慣是 `kubectl get pods` $
ightarrow$ 看到 `Running` $
ightarrow$ 認為服務正常。

但對於 SRE 來說，這是一種危險的錯覺。Kubernetes 的狀態更新是一個**非同步的宣告式狀態機**。API Server 記錄的是「期望狀態」，而 Kubelet 負責將其轉化為「實際狀態」。在這兩者之間，存在著一個巨大的 **SRE 資訊差 (SRE Gap)**。

## 1. 從「意圖」到「過程」：Pod 的垂直路徑

一個 Pod 從被定義到真正運行，經歷了三個層級的交接。如果你只看 API，你只能看到結果，而看不到過程中的崩潰。

### 第一階段：控制平面 (Orchestration Layer)
- **API Server**：接收 YAML $
ightarrow$ 寫入 etcd $
ightarrow$ 狀態：`Pending`。
- **Scheduler**：發現未分配的 Pod $
ightarrow$ 根據資源權重選擇節點 $
ightarrow$ 寫回 `Binding` 對象。
- **真相**：此時 Pod 依然是 `Pending`，它僅僅是一個「被安排好位置的願望」。

### 第二階段：節點實現 (Runtime Layer)
當 Kubelet 發現 Pod 被分配到自己節點後，真正的「肉搏戰」開始了：
1. **建立 Sandbox**：呼叫 CRI (如 containerd) 啟動 `pause` 容器。這決定了 Pod 的網路命名空間與 Cgroup 父級。
2. **拉取鏡像**：呼叫 `PullImage`。如果鏡像太大或 Registry 慢，Pod 會卡在 `ContainerCreating`。
3. **CNI 佈線**：呼叫 CNI 插件 (如 Calico/Cilium) 分配 IP 並創建 `veth pair`。如果 IPAM 耗盡，這裡會直接超時。
4. **執行容器**：呼叫 `CreateContainer` $
ightarrow$ `StartContainer` $
ightarrow$ `runc` 執行 `clone()`。

- **真相**：當你看到 `ContainerCreating` 時，故障可能發生在任何一個子步驟。API Server 只會告訴你「還在創建中」，但具體是鏡像拉不下來、還是 IP 分配失敗，你必須下到節點去看 `journalctl -u kubelet`。

### 第三階段：反饋迴路 (Observability)
CRI 報告容器已啟動 $
ightarrow$ Kubelet 更新 API 狀態為 `Running`。

- **真相**：`Running` 僅僅意味著進程已被核心啟動，並不代表應用程式已經準備好接收流量 (除非你有正確的 `Readiness Probe`)。

---

## 2. SRE 除錯矩陣：不要被 API 蒙蔽

當 Pod 狀態異常時，請停止詢問 API，直接向下探索「地形」：

| API 狀態 | 可能的故障點 | 垂直層級 | 診斷指令 | 核心真相 |
| :--- | :--- | :--- | :--- | :--- |
| **`Pending`** | 調度器 / 資源不足 | 編排層 | `kubectl describe pod` | 節點沒空間了 |
| **`ContainerCreating`** | CRI (鏡像) / CNI (網路) | 運行時層 | `journalctl -u kubelet` | 鏡像拉取超時或 CNI 插件崩潰 |
| **`CrashLoopBackOff`** | 入口點故障 / 核心 OOM | 核心層 | `kubectl logs --previous` | 應用程式崩潰或觸發 Cgroup 記憶體限制 |
| **`ErrImagePull`** | 認證失敗 / 網路不通 | 運行時層 | `crictl pull <image>` | Registry 憑據過期 |

## 3. SRE 避坑指南：那些隱形的生命週期陷阱

- **`pause` 容器的崩潰**：如果 `pause` 容器被殺掉，整個 Pod 的網路命名空間會立即毀滅，導致所有 Sidecar 和主容器瞬間斷網。
- **CNI 延遲**：在超大規模叢集中，CNI 分配 IP 的時間可能超過 Kubelet 的超時閾值，導致 Pod 陷入 `ContainerCreating` $
ightarrow$ `Terminating` 的無限循環。
- **殭屍 Pod (Zombie Pods)**：有時 Pod 在 API 中已刪除，但核心進程依然在運行，因為 `TearDownPodSandbox` 呼叫失敗。這會導致資源洩漏且無法重新調度。

## 結語：地圖 $
eq$ 地形

API Server 提供的狀態是「地圖 (The Map)」，而節點上的進程狀態才是「地形 (The Terrain)」。

作為 SRE，當你發現地圖與現實不符時，唯一正確的做法就是**向下鑽取 (Drill-down)**：從 `kubectl` $
ightarrow$ `journalctl` $
ightarrow$ `crictl` $
ightarrow$ `dmesg`。只有觸碰到核心的 `task_struct`，你才能掌握真正的真相。

---
**技術關鍵字：**
- Pod Sandbox (Pod 沙箱)
- Kubelet SyncLoop (同步迴路)
- CRI (Container Runtime Interface)
- CNI (Container Network Interface)
- a liveness/readiness probe (探針)
- Pod Lifecycle (Pod 生命週期)