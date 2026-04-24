# 為什麼你的 Pod 一直卡在 ContainerCreating？：揭秘 K8s 儲存底層的「掛載地獄」

在 Kubernetes 的世界裡，儲存 (Storage) 往往是最容易讓人崩潰的部分。你定義了 PVC，建立了 StorageClass，但 Pod 卻像進入了黑洞一樣，永遠卡在 `ContainerCreating`。

大多數人習慣執行 `kubectl describe pod` 看到 `MountVolume.SetUp failed` 就開始猜測是權限問題或雲端配額不足。但作為 SRE，我們必須意識到：**K8s 的 Volume 並不是一個「磁碟」，而是一個被精確管理生命週期的「掛載點 (Mount Point)」。**

從一個 PVC 請求變成 Pod 裡一個可以寫入的目錄，封包和指令經歷了一場從控制平面 $ightarrow$ CSI 驅動 $ightarrow$ Kubelet $ightarrow$ Linux 核心 VFS 的長途跋涉。

## 儲存請求的「垂直路徑」：從 API 到 Block

### 1. 宣告層 (K8s API)
一切始於 `PersistentVolumeClaim` (PVC)。這是一個「願望清單」，告訴 K8s 我需要多少空間、什麼樣的存取模式 (`ReadWriteOnce` 等)。
- **SRE 訊號**：如果 PVC 狀態一直是 `Pending`，請直接檢查 `StorageClass` 設定或雲端供應商的 Quota，不要在 Pod 層面浪費時間。

### 2. 編排層 (Kubelet & CSI)
當 Pod 被調度到節點後，真正的「搬運」開始了：
1. **Attach (附加)**：控制平面通知 CSI 驅動，將雲端磁碟（如 AWS EBS）附加到該 VM 實例。
2. **Mount (掛載)**：主機上的 `Kubelet` 呼叫 CSI Node Plugin，將遠端磁碟掛載到主機的一個全域目錄（例如 `/var/lib/kubelet/plugins/...`）。
3. **Bind Mount (綁定掛載)**：這是最關鍵的一步。Kubelet 將上述全域目錄**再次綁定掛載**到 Pod 的特定路徑中。

- **SRE 訊號**：`MountVolume.SetUp failed` 通常意味著 CSI 驅動無法連接後端儲存，或者磁碟被之前的節點「佔住」了（典型的 Multi-Attach 錯誤）。

### 3. 核心層 (VFS & Kernel)
最後，Linux 核心接管 I/O。無論底層是 XFS, ext4 還是 NFS，對應用程式來說都經過 **VFS (Virtual File System)**。
- **SRE 訊號**：如果你在日誌中看到 `EIO (Input/Output error)`，這通常是驅動層級的災難（例如 NFS 網路超時或硬體損毀），與 K8s 配置無關。

---

## SRE 除錯矩陣：從現象到原點

| 現象 | K8s 訊號 | 核心/主機原語 | 推薦工具 |
| :--- | :--- | :--- | :--- |
| **Pod 永久掛起** | `ContainerCreating` | `dmesg` (Timeout) | `kubectl describe` $ightarrow$ `dmesg` |
| **檔案系統唯讀** | `Read-only file system` | `mount -o remount,rw` | `mount \| grep <path>` |
| **NFS 句柄失效** | `Stale NFS file handle` | `nfsstat` / `rpcdebug` | `df -h` (會直接卡死) |
| **I/O 極慢** | 高 `iowait` | `bio` 請求隊列 | `iostat -xz 1` / `biotop` |

## 避坑指南：那些隱形的儲存陷阱

1. **儲存「殭屍」 (Orphaned Mounts)**：
   有時候 Pod 刪除了，但核心的掛載點沒被清理掉。這會導致新 Pod 嘗試掛載同一個 Volume 時發生衝突。
2. **掛載傳播 (Mount Propagation)**：
   如果 CSI 驅動的 `mountPropagation` 沒有設定為 `Bidirectional`，驅動容器內建立的掛載對主機或 Pod 來說是不可見的。
3. **根分區壓力**：
   如果主機根分區滿了，Kubelet 可能無法建立 bind-mount 所需的目錄結構，導致 Volume 掛載失敗。

## 結語：不要只看 `kubectl`

面對儲存故障，最強大的工具不是 `kubectl get pvc`，而是**直接登入節點**。

檢查 `dmesg` 看核心錯誤 $ightarrow$ 檢查 `mount` 確認綁定鏈 $ightarrow$ 檢查 `iostat` 確認後端響應。當你能跳出 K8s 的抽象層，直接面對 Linux 核心的 VFS 時，儲存問題才真正變得可控。

---
**技術關鍵字：**
- CSI (Container Storage Interface)
- Bind Mount (綁定掛載)
- VFS (Virtual File System)
- Multi-Attach Error (多重附加錯誤)
- Mount Propagation (掛載傳播)
