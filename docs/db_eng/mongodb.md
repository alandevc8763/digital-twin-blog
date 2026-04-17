# MongoDB：資料備份與還原實踐 (Dump & Restore)

## Taxonomy
- **Domain**: Database Engineering / Disaster Recovery
- **Signal**: 🥈 Silver (Operational Standard)

## Core Insight
資料備份是資料庫維運的最後一道防線。`mongodump` 與 `mongorestore` 構成了 MongoDB 基礎備份的閉環。關鍵在於理解 **BSON 格式** 的高效性以及在大規模資料恢復時的 **性能影響**。

## Implementation Details

### 📦 備份流程 (mongodump)
`mongodump` 將資料導出為 BSON 格式，確保資料類型的原生保留。

**常用指令組合：**
- **全庫備份**：`mongodump --db <db_name>`
- **特定集合備份**：`mongodump --db <db_name> --collection <coll_name>`
- **遠端伺服器備份**：使用 URI 格式 `mongodump --uri="mongodb://user:pass@host:port/db"`

### 🔄 還原流程 (mongorestore)
`mongorestore` 負責將 BSON 檔案還原至目標庫。

**關鍵指令：**
- **全庫還原**：`mongorestore --db <db_name> <backup_path>`
- **特定集合還原**：`mongorestore --db <db_name> --collection <coll_name> <path_to_bson>`

### ⚠️ 運維坑洞
- **性能衝擊**：在生產環境執行 `mongodump` 可能導致 I/O 飆高。建議在從庫 (Secondary) 執行或在低峰期操作。
- **空間壓力**：BSON 格式相對較大，還原時需確保目標磁碟有足夠緩衝空間。
- **版本兼容**：確保 `mongodump` 與 `mongorestore` 的版本與資料庫版本一致，避免 BSON 格式不兼容。

## Synergy
- **SRE 自動化**：建議將此流程封裝進 CronJob 或 Kubernetes Job，實現每日自動快照。
- **Cloud Migration**：這是將資料從 On-premise 遷移至 MongoDB Atlas 的最快路徑。
