# 別把 NAS 當成儲存盒：ZFS 與數據完整性的真相
很多人對 NAS 的認知還停留在「買個機殼 $\rightarrow$ 插滿硬碟 $\rightarrow$ 設定 RAID $\rightarrow$ 儲存檔案」。在這種認知下，NAS 就像是一個巨大的儲存桶，只要 RAID 顯示 `Healthy`，數據就是安全的。
但對 SRE 來說，這是最危險的錯覺。
真正的儲存挑戰不在於「硬碟壞了怎麼辦」（這是 20 年前的問題），而是在於 **Silent Data Corruption (靜默數據損毀)**，也就是俗稱的 **Bit Rot**。當你的 RAID 顯示健康，但打開一個 5 年前的 PDF 卻提示「檔案損毀」時，你才會意識到，傳統的 RAID 其實根本沒在保護你的數據。
## 硬件層：被忽視的「地雷」
在討論軟體之前，硬件層有兩個最容易掉進去的坑。
### 1. SMR 硬碟：RAID 的噩夢
如果你在買硬碟時沒區分 **CMR (Conventional Magnetic Recording)** 和 **SMR (Shingled Magnetic Recording)**，你可能已經埋下了炸彈。
SMR 為了增加容量，將數據像瓦片一樣重疊寫入。這導致它在進行大量隨機寫入或 RAID 重構（Rebuild）時，會產生極其嚴重的寫入放大。我見過太多人在 Rebuild 時因為 SMR 的性能崩潰，導致重建時間從 24 小時拉長到一週，甚至直接導致另一顆硬碟崩潰而全盤丟失。
### 2. ECC RAM：ZFS 的保命符
很多人覺得家用伺服器不需要 ECC 記憶體，但如果你用 ZFS，這不是選項，而是必要條件。
ZFS 的 ARC (Adaptive Replacement Cache) 會在記憶體中快取大量的 metadata 和數據。如果記憶體發生一次 Bit-flip（位元翻轉），而你沒有 ECC 糾錯，ZFS 可能會將這個錯誤的數據「忠實地」寫入硬碟。這就變成了一場悲劇：你的文件系統在努力地幫你把損毀的數據永久化。
## 文件系統：從「被動儲存」到「主動修復」
為什麼我強烈建議放棄傳統硬件 RAID 而選擇 ZFS？因為 ZFS 將儲存從一個「被動的桶」變成了一個「主動的完整性引擎」。
### Copy-on-Write (CoW) 與寫入洞 (Write Hole)
傳統文件系統在更新數據時是直接覆蓋（Overwrite）。如果寫到一半斷電，就可能出現數據半新半舊的 `Write Hole` 狀態。
ZFS 採取 **CoW (Copy-on-Write)**：它永遠不會覆蓋原數據，而是先將新數據寫到新區塊，確認寫入成功後才更新指針。這意味著無論什麼時候斷電，你的數據要麼是舊的，要麼是新的，但絕對不會是「損毀的」。
### ZFS VDEVs：放棄 RAID 5 的執念
在設計冗餘時，很多人迷信 RAID 5 (Z1)。但在大容量硬碟時代，Z1 的風險太高。在 Rebuild 過程中，任何一次不可恢復的讀取錯誤 (URE) 都會導致整個陣列崩潰。
**我的建議是 Z2 (雙拋錯)**。對於家庭伺服器，Z2 是穩定性與容量的最佳平衡點；如果你在存儲極其重要的歷史檔案，直接上 Z3。
## 實戰分析：對抗 Bit Rot
讓我們回到最開始的噩夢：檔案損毀但 RAID 顯示正常。
**傳統 RAID 的邏輯**：
- 監控硬碟的 S.M.A.R.T 狀態。
- 硬碟沒死 $\rightarrow$ 數據沒錯。
- $\rightarrow$ **結果**：完全無法發現磁介質老化導致的 Bit-flip。
**ZFS 的邏輯**：
- 每個數據塊都攜帶一個 **Checksum (校驗碼)**。
- 當你讀取檔案時，ZFS 會計算當前數據的校驗碼，並與儲存的校驗碼比對。
- 如果比對失敗 $\rightarrow$ ZFS 知道數據壞了 $\rightarrow$ 立即從鏡像或奇偶校驗塊中提取正確數據 $\rightarrow$ **自動修復** 損毀的塊。
這個過程在 ZFS 中稱為 `scrub`。如果你不定期執行 `zpool scrub`，你的 ZFS 也就失去了靈魂。
## 總結：思考維度的轉移
請停止思考「容量」，開始思考「完整性」。
一個沒有校驗機制的文件系統，無論容量多大、RAID 等級多高，在本質上都只是一個隨時會爆炸的數據炸彈。真正的儲存方案，應該是在數據損毀發生的那一刻，系統能告訴你：「發現一個損毀塊，但我已經幫你修好了。」


---

## 🔗 Related Insights

- Comparing this integrity-first approach with DAS raw speed: [Nas Vs Das](nas-vs-das.md)
- Deep dive into the VFS path ZFS uses for checksumming: [Vfs Tracing](../performance/vfs-tracing.md)
