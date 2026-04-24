---
title: "Vfs Tracing"
date: 2026-04-24
categories: ["SRE"]
tags: ["SRE", "Performance", "Vfs-Tracing.Md"]
draft: false
---

# 隱藏的 I/O 稅：VFS 與 Page Cache 的追蹤實踐
當你調用 `read()` 讀取一個檔案時，你以為數據直接從磁碟來到記憶體？
不，它經歷了一場極其複雜的旅程。
## VFS：文件系統的「外交官」
VFS (Virtual File System) 是 Linux 內核的一層抽象。無論底層是 Ext4、XFS 還是 NFS，對應用程序來說都一樣。但這個抽象層是有成本的。
### 1. Dentry Cache：名稱的陷阱
每次打開檔案，內核都要將路徑（如 `/var/log/syslog`）轉換為 Inode 編號。這個過程依賴於 **Dentry Cache**。如果你的系統有數百萬個小檔案且快取不足，你會發現 CPU 消耗在 `dentry_lookup` 上，而磁碟 IOPS 卻很低。
### 2. Page Cache：最強的加速器，也是最大的坑
Linux 會利用幾乎所有剩餘記憶體作為 Page Cache。
- **Read-ahead (預讀)**：內核猜測你要讀接下來的塊，提前加載。
- **Write-back (回寫)**：`write()` 調用幾乎總是瞬間完成，因為它只寫到了記憶體。
**SRE 的噩夢：Write-out Spike**
當記憶體中的髒頁 (Dirty Pages) 過多，觸發內核的 `pdflush` 強制回寫時，整個系統的 I/O 會突然被佔滿，導致所有讀寫操作卡死。這就是為什麼有些系統會莫名其妙地出現數秒的「凍結」。
## 實戰：用 eBPF 穿透 VFS
當 `iostat` 告訴你磁碟很慢，但你懷疑是內核層的問題時，請使用 eBPF。
使用 `bpftrace` 追蹤 `vfs_read` 的延遲分佈：
`bpftrace -e 'kprobe:vfs_read { @start[tid] = nsecs; } kretprobe:vfs_read /@start[tid]/ { @latency = hist(nsecs - @start[tid]); delete(@start[tid]); }'`
這能讓你一眼看出：延遲是發生在 **VFS 層 (軟體開銷)** 還是 **Block Layer (硬體開銷)**。
## 總結：I/O 優化的終點是避開 VFS
如果你追求極限性能，目標應該是 **「減少 syscall 次數」** 和 **「繞過 VFS」**。
- 使用 `mmap` 減少數據拷貝。
- 使用 `io_uring` 實現真正的異步 I/O，消除 syscall 損耗。
**真正的 I/O 專家不看磁碟速度，他們看的是 Page Cache 的命中率與 syscall 的頻次。**


---

## 🔗 Related Insights

- How ZFS handles these VFS operations to ensure integrity: [Nas Zfs Integrity](../homelab/nas-zfs-integrity.md)
- Correlating VFS latency with system-wide PSI pressure: [Psi Metrics](psi-metrics.md)