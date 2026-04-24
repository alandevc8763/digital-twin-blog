---
title: "Proxmox VE: HomeLab 中的超融合基礎設施 (HCI) 實踐"
date: 2026-04-24
categories: ["Homelab"]
tags: ["Homelab", "Proxmox", "Virtualization", "HCI"]
draft: false
---

# Proxmox VE：將家用電腦轉化為微型數據中心

對於大多數 HomeLab 玩家來說，虛擬化可能是為了「省錢」——用一台強大的主機運行十幾個服務。但從 SRE 的視角來看，Proxmox VE (PVE) 的真正價值在於它將 **KVM (全虛擬化)** 與 **LXC (容器虛擬化)** 完美地整合在單一的管理平面中，並原生支持 ZFS 與 Ceph，讓家用硬件也能跑出「企業級」的超融合基礎設施 (HCI)。

---

## 1. 虛擬化策略：KVM 與 LXC 的「隔離 vs 性能」權衡

在 PVE 中部署服務時，最容易犯的錯誤就是所有東西都用 VM (KVM)。實際上，正確的選型應該基於對 **Virtualization Overhead** 的精確控制。

### KVM (Kernel-based Virtual Machine)：絕對的隔離
KVM 是完整的硬件虛擬化，每個 VM 擁有獨立的內核。
- **場景**：當你需要運行 Windows、BSD，或者需要對內核進行深度調試時，KVM 是唯一選擇。
- **代價**：Guest OS 稅。即使是一個簡單的 Linux VM，也會消耗額外的 RAM 和 CPU 來維護其內核。

### LXC (Linux Containers)：近乎原生的性能
LXC 是操作系統級虛擬化，與宿主機共享內核。
- **場景**：部署 Web 服務、微服務、輕量級 Bot。
- **優勢**：秒級啟動，內存占用極低。如果你運行一個簡單的 Nginx，LXC 的資源開銷幾乎可以忽略不計。

**SRE 決策模型**：$\text{需要不同內核 / 高安全性} \rightarrow \text{KVM}$；$\text{追求性能 / 低開銷} \rightarrow \text{LXC}$。

---

## 2. 存儲演進：從本地 ZFS 到分佈式 Ceph

PVE 的強大在於它對存儲層的深度集成。

### ZFS：本地存儲的定海神針
在單機 PVE 中，ZFS 是最佳選擇。它提供的 **快照 (Snapshot)** 和 **克隆 (Clone)** 功能讓 SRE 能在進行風險操作前秒級備份。更強大的是 `zfs send/receive`，可以在兩台 PVE 節點之間實現高效的 VM 異地同步。

### Ceph：實現真正的高可用 (HA)
當你的 HomeLab 擴展到多台主機時，Ceph 讓「超融合」真正落地。Ceph 將數據分片並分佈在所有節點上。
- **核心能力**：如果 Node A 突然宕機，只要 Ceph 存儲層依然健康，PVE 的 HA 機制會立即在 Node B 上重啟該 VM。
- **體驗**：這將存儲從「綁定在某台主機上」變成了「集群共享資源」。

---

## 🚨 SRE 調試實錄：消失的 Quorum 與「腦裂」危機

在構建 PVE 集群時，最讓新手崩潰的場景莫過於 **「兩節點集群的死鎖」**。

**現象**：你部署了一個 2 節點集群，當其中一台主機因斷電關機後，發現剩餘的那台主機雖然運行正常，但所有 VM 突然被鎖定，無法啟動或修改。

**底層邏輯 (The Quorum Rule)**：
PVE 通過 Corosync 維護集群狀態。為了防止 **「腦裂 (Split Brain)」**（即兩台主機都認為自己是 Master 並同時寫入同一塊磁碟導致數據損毀），PVE 嚴格執行 $\text{n/2 + 1}$ 的投票機制。
在 2 節點集群中，失效一台後，剩餘節點只有 1 票 (1/2)，未達到過半數 $\rightarrow$ **失去 Quorum (法定人數)** $\rightarrow$ 進入唯讀模式。

**解決方案**：
不要嘗試強行修改集群狀態，最優解是引入 **QDevice (Quorum Device)**。你可以用一台極低功耗的設備（如 Raspberry Pi 或 甚至是一個小 VM）作為第三個投票者。它不存儲數據，僅在投票時提供決定性的那一票，從而確保集群在掉一台節點時依然能正常運行。

---

## 總結：從 PC 到 Data Center

Proxmox VE 的意義在於它降低了接觸 enterprise-grade infra 的門檻。透過在 KVM/LXC 之間做權衡，在 ZFS/Ceph 之間選擇擴展路徑，以及正確處理 Quorum 機制，你實際上是在家裡模擬一個真實的數據中心運行環境。

---
**參考來源：**
- [Proxmox VE Official Wiki](https://pve.proxmox.com/wiki/Main_Page)
- 第二大腦 $\rightarrow$ `wiki/homelab/proxmox-deep-dive.md` (PVE 超融合基礎設施分析)
- [Proxmox Forum: Quorum and QDevice Guide](https://forum.proxmox.com/)
