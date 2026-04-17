# Kafka 快速操作指南：從部署到調試

## Taxonomy
- **Domain**: Cloud Operations / SRE
- **Signal**: 🥉 Bronze (Operational Tooling)

## Core Insight
對於 SRE 而言，掌握 Kafka 的 CLI 工具不僅是為了部署，更是為了在故障發生時快速定位問題。理解 `kafka-topics` $\rightarrow$ `kafka-console-producer` $\rightarrow$ `kafka-console-consumer` 的數據流向是所有調試的基礎。

## Implementation Details

### 🛠️ 核心指令速查表

#### 1. Topic 管理 (The Skeleton)
- **創建 Topic**：
  `kafka-topics.sh --create --topic my-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 2`
- **查看 Topic 詳情**：
  `kafka-topics.sh --describe --topic my-topic --bootstrap-server localhost:9092`
- **刪除 Topic**：
  `kafka-topics.sh --delete --topic my-topic --bootstrap-server localhost:9092`

#### 2. 數據生產與消費 (The Flow)
- **快速發送消息 (Producer)**：
  `kafka-console-producer.sh --topic my-topic --bootstrap-server localhost:9092` $\rightarrow$ *進入交互模式後輸入消息*
- **從頭讀取所有消息 (Consumer)**：
  `kafka-console-consumer.sh --topic my-topic --from-beginning --bootstrap-server localhost:9092`
- **指定消費者組讀取 (Consumer Group)**：
  `kafka-console-consumer.sh --topic my-topic --group my-group --bootstrap-server localhost:9092`

#### 3. 狀態監控 (The Health Check)
- **查看消費者組偏移量 (Consumer Lag)**：
  `kafka-consumer-groups.sh --describe --group my-group --bootstrap-server localhost:9092`
  *$\text{關鍵指標}$：$\text{LOG-END-OFFSET} - \text{CURRENT-OFFSET} = \text{LAG}$*

### ⚠️ SRE 實操坑洞
- **Zookeeper 依賴**：在舊版本中，所有元數據都在 ZK，如果 ZK 響應慢，會導致所有 Broker 劇烈抖動。
- **Partition 數量不可減少**：一旦 Topic 創建，Partition 數量只能增加不能減少。建議在創建時根據預期吞吐量合理設定。

## Synergy
- **Prometheus Monitoring**：建議將 `kafka-exporter` 接入 Prometheus，將上述 CLI 指標轉化為實時 Dashboard 監控。
- **Log Aggregation**：Kafka 是 ELK 棧 (Elasticsearch-Logstash-Kibana) 中最核心的緩衝層。
