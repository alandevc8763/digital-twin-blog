---
title: "NAS ZFS Truth"
date: 2026-04-24
categories: ["Homelab"]
tags: ["Homelab", "ZFS", "Data-Integrity"]
draft: false
---

# NAS 的真相：為什麼你需要的不是「容量」，而是「完整性」？

## 核心視角：從「被動桶子」到「主動引擎」
大多數人看待 NAS 時，首先想到的是「容量」——多少 TB 的空間能存我的電影和照片。但對於 SRE 來說，這是一個危險的誤區。

專業的 NAS 不應該被視為一個「裝滿硬碟的盒子」，而應該是一個旨在最大化 **數據持久性 (Data Durability)** 與 **可用性 (Availability)** 的系統。真正的分水嶺在於你使用的是傳統的硬件 RAID，還是像 **ZFS** 這樣具備「自癒能力」的軟件定義存儲。

---

## 1. 物理層的陷阱：CMR vs SMR
在購買硬碟前，你必須面對一個隱形的陷阱：**SMR (疊瓦式磁記錄)**。
SMR 硬碟為了增加密度，將數據像瓦片一樣重疊寫入。這在單純的存檔時沒問題，但在 RAID 重構 (Rebuild) 過程中，SMR 的寫入放大效應會導致重構速度慢到令人絕望，甚至直接導致整個陣列崩潰。

**SRE 建議**：永遠選擇 **CMR (傳統磁記錄)** 硬碟，並使用 **IT Mode (Initiator Target)** 的 HBA 卡將原盤直接交給操作系統，徹底擺脫私有 RAID 卡的廠商鎖定。

## 2. ZFS：數據完整性的黃金標準
為什麼 ZFS 被稱為存儲界的「黃金標準」？因為它解決了傳統存儲最恐懼的問題：**靜默數據損壞 (Silent Data Corruption / Bit Rot)**。

### Copy-on-Write (CoW)：消除寫入空洞
傳統文件系統在更新數據時會覆蓋原位。如果在此過程中斷電，會產生所謂的「寫入空洞 (Write Hole)」，導致文件損毀。ZFS 採用 **CoW** 機制：永遠不覆蓋原數據，而是先將新數據寫入新塊，最後才更新指針。這意味著你的數據在寫入完成前，原件永遠是安全的。

### ARC 與 ZIL：性能與安全的分離
ZFS 並不依賴傳統的 LRU 緩存，而是使用 **ARC (Adaptive Replacement Cache)**，同時追蹤數據的「頻率」與「近期性」，讓緩存命中率大幅提升。而對於同步寫入，則通過 **ZIL (ZFS Intent Log)** 確保在不犧牲性能的前提下，數據能立即持久化。

## 3. 冗餘策略：RAID-Z 的權衡
選擇冗餘方案時，你是在 $\text{容量}$ 與 $\text{安全性}$ 之間做交易：

- **Mirroring (RAID 1/10)**：性能最強，重建最快，但空間損耗 50%。適合存放頻繁讀寫的虛擬機鏡像。
- **RAID-Z1 (1 盤失效)**：空間利用率高，但在重建期間如果再掉一顆盤，所有數據全部消失。
- **RAID-Z2 (2 盤失效)**：HomeLab 的「甜蜜點」。兼顧了容量與安全性，允許同時損壞兩顆盤而數據不丟失。

---

## 🚨 SRE 調試：面對「位翻轉 (Bit Rot)」危機
想像一個場景：你的文件系統顯示「健康」，但當你打開一張五年前的照片時，發現圖片中間出現了一條詭異的色塊。這就是 **Bit Rot**。

傳統 RAID 只能偵測到「硬碟壞了」，但無法偵測到「數據錯了」。ZFS 的解決方案是：**為每個數據塊計算校驗和 (Checksum)**。

當你執行 `scrub` (週期性掃描) 時，ZFS 會將實際讀出的數據與存儲的校驗和對比。一旦發現不匹配，ZFS 會利用冗餘副本自動獲取正確的數據，並**在後台自動修復**損毀的數據塊。這就是所謂的「自癒 (Self-Healing)」。

## 總結
停止思考「我能存多少」，開始思考「我的數據是否正確」。一個沒有校驗和文件系統 (ZFS/Btrfs) 的 NAS，本質上就是一顆隨時會爆炸的靜默損壞定時炸彈。

---
**參考來源：**
- [OpenZFS Documentation](https://openzfs.github.io/openzfs-docs/)
- 第二大腦 $\rightarrow$ `wiki/homelab/nas-fundamentals.md` (NAS 存儲架構與數據完整性)
- [ZFS Guide by FreeBSD](https://www.freebsd.org/doc/articles/zfs-guide/)
