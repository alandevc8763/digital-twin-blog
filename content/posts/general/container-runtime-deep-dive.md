---
title: "Container Runtime Deep Dive"
date: 2026-04-24
categories: ["General"]
tags: ["General", "Container-Runtime-Deep-Dive.Md"]
draft: false
---

# 解構容器運行時：從 Kubelet 意圖到二進制執行的完整路徑

如果你問一個開發者「容器是什麼？」，他可能會回答「一個打包了環境的鏡像」。但如果你問一個 SRE「容器是如何啟動的？」，答案將會是一場複雜的系統呼叫交響曲。

很多工程師以為 `docker run` 或 `kubectl apply` 之後，容器就直接「跑起來」了。事實上，從 K8s 的宣告式意圖到核心執行二進制文件，中間經過了一套由 **OCI 標準** 治理的模塊化鏈路。

## 1. 權責分離：高級運行時 vs 低級運行時

現代容器棧不再是一個單體，而是分為兩層：

- **高級運行時 (High-Level Runtime)**：如 `containerd` 或 `CRI-O`。它們是「管理員」，負責拉取鏡像、管理存儲、設置網路以及與 Kubelet 通信。它們不直接創建容器，而是編排資源。
- **低級運行時 (Low-Level Runtime)**：如 `runc`。它是「工人」，一個短暫的進程。它的唯一目標是：與 Linux 核心交互，創建命名空間、設置控制組 (Cgroups)、切換根目錄，然後執行目標程序。

這就是 OCI (Open Container Initiative) 的核心價值：只要符合標準，你可以隨意替換 `runc` 為 `Kata Containers` (輕量級 VM) 或 `gVisor` (用戶態內核)，而無需修改 K8s。

## 2. 啟動之舞：`runc` 的執行生命週期

容器的啟動並非單一事件，而是一系列精確的系統呼叫順序。當 `containerd` 要求啟動容器時，`runc` 會執行以下「舞蹈」：

### 第一步：分叉與克隆 (Fork & Clone)
`runc` 不使用簡單的 `fork()`，而是使用 `clone()` 並帶上特定的標誌 (Flags)：
- `CLONE_NEWNET` $
ightarrow$ 創建私有網路棧。
- `CLONE_NEWPID` $
ightarrow$ 讓進程以為自己是 `PID 1`。
- `CLONE_NEWNS` $
ightarrow$ 隔離掛載點。
這決定了容器的「視角」隔離。

### 第二步：資源枷鎖 (Cgroup Handshake)
在執行應用之前，必須先限制它的能力。`runc` 將新進程的 PID 寫入 `/sys/fs/cgroup/.../cgroup.procs`。至此，核心的 CFS 調度器和記憶體管理模組開始對該進程實施配額管控。

### 第三步：根目錄切換 (`pivot_root`)
為了防止容器看到主機的文件系統，`runc` 使用 `pivot_root` 將根目錄切換到鏡像解壓後的目錄。
**SRE 坑點**：`pivot_root` 要求新根目錄必須是一個「掛載點」。如果你手動執行 `runc` 且路徑只是個普通目錄，會直接報 `EINVAL (Invalid Argument)`。

### 第四步：最終轉化 (`execve`)
環境鎖定完成後，`runc` 呼叫 `execve()`。這會用目標應用程序的二進制文件替換掉 `runc` 進程本身。

---

## 3. 穩定錨點：`containerd-shim` 的必要性

如果你觀察 `ps -ef`，你會發現應用進程的父進程不是 `containerd`，而是一個叫 `containerd-shim` 的小程序。為什麼需要這個中間層？

**Shim 是容器生命週期的「穩定錨點」。**

- **解耦生命週期**：如果沒有 Shim，`containerd` 必須維持所有容器的標準輸出/錯誤管道。如果 `containerd` 崩潰或升級重啟，所有容器都會跟著死掉。有了 Shim，`containerd` 重啟後可以重新連接到 Shim，而容器毫髮無傷。
- **僵屍進程回收**：Shim 負責 `waitpid`，確保容器退出後能正確記錄退出碼並清理資源，防止主機出現大量 `<defunct>` 僵屍進程。

## 4. SRE 診斷矩陣：當容器啟動失敗時

| 症狀 | 潛在位置 | 核心訊號 | 診斷工具 |
| :--- | :--- | :--- | :--- |
| **`ContainerCreating` 永久掛起** | 高級運行時 (CRI) | `TaskService` gRPC 錯誤 | `journalctl -u containerd` |
| **`CreateContainerError`** | 低級運行時 (`runc`) | `pivot_root: invalid argument` | `runc` 執行日誌 |
| **`Running` 但無日誌/不響應** | 穩定錨點 (Shim) | PPID 為 1 (Shim 已死) | `ps -ef` $
ightarrow$ 檢查父進程 |
| **啟動速度極慢** | 核心/儲存層 | `mount` 系統呼叫延遲 | `strace` $
ightarrow$ `sys_mount` |

## 結語：從「黑盒」到「透明」

容器並不是什麼神奇的封裝，它只是 Linux 核心功能的巧妙組合。從 Kubelet 的意圖 $
ightarrow$ CRI 的編排 $
ightarrow$ Shim 的錨定 $
ightarrow$ `runc` 的執行 $
ightarrow$ 核心的強制執行。

理解這一路徑，能讓你從「根據日誌猜測」轉化為「根據系統呼叫斷定」故障根因。

---
**技術關鍵字：**
- OCI (Open Container Initiative)
- containerd / CRI-O (高級運行時)
- runc (低級運行時)
- containerd-shim (生命週期錨點)
- pivot_root (根目錄切換)
- execve (程序替換)