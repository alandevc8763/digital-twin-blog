---
title: "Core Concepts"
date: 2026-04-24
categories: ["General"]
tags: ["General", "Cloud Infra", "Kafka", "Core-Concepts.Md"]
draft: false
---

# Kafka 核心概念與元件詳解

## Taxonomy
- **Domain**: Distributed Systems / Message Queue
- **Signal**: 🥇 Gold (Foundational Knowledge)

## Core Insight
Kafka 的強大在於它將訊息系統抽象為一個 **「分佈式提交日誌 (Distributed Commit Log)」**。透過將 Topic 拆分為 Partition，並使用 Offset 作為消費游標，Kafka 實現了極高吞吐量的順序寫入與並行消費，從而解耦了生產者與消費者的處理速度。

## 核心元件詳解

### 1. Topic (主題)
Topic 是 Kafka 中對訊息進行邏輯分類的單位。
- **特性**：一個 Topic 可以包含多個 Partition。
- **設計要點**：Topic 的命名應反映其數據的語義（例如 `user_login_events`），且一旦創建，Partition 數量增加後通常無法減少。

### 2. Partition (分區) $\rightarrow$ 並行度的核心
Partition 是 Topic 的物理拆分，是 Kafka 實現水平擴展的基礎。
- **分發邏輯**：生產者根據 Key 的 Hash 值決定訊息進入哪個 Partition。若 Key 為空，則採用輪詢 (Round-robin) 策略。
- **順序保證**：Kafka 僅保證 **單個 Partition 內** 的訊息順序，不保證整個 Topic 的全局順序。

### 3. Message/Record (訊息)
訊息是 Kafka 傳輸的最小原子單位。
- **組成**：`Key` (用於分區), `Value` (實際數據), `Timestamp` (時間戳), `Headers` (元數據)。
- **持久化**：訊息以 append-only 的方式寫入日誌文件，確保了極高的寫入性能。

### 4. Offset (偏移量) $\rightarrow$ 消費進度指標
Offset 是訊息在 Partition 中的唯一序列號，從 0 開始遞增。
- **消費機制**：消費者記錄自己的 Offset，從而實現斷點續傳。
- **提交策略**：支持手動提交或自動提交。Offset 的精確管理決定了訊息是「至多一次」還是「至少一次」交付。

### 5. Broker (代理伺服器)
Broker 是 Kafka 集群中的一個節點，負責存儲數據與處理請求。
- **職責**：處理生產者的寫入、消費者的讀取，以及維護 Partition 的副本 (Replica) 同步。
- **高可用**：通過 Leader/Follower 機制，當 Leader Broker 故障時，由 ISR (In-Sync Replicas) 中選出新 Leader。

### 6. Producer & Consumer (生產者與消費者)
- **Producer**：決定數據的分發策略，通過 `acks` 參數控制寫入的可靠性（0, 1, all）。
- **Consumer Group**：多個消費者組成一個 Group，共同分擔一個 Topic 的 Partition 處理工作。**一個 Partition 在同一時間只能由同一個 Group 內的一個消費者處理**，這保證了消費的順序性。

### 7. Coordinator (協調者: Zookeeper / KRaft)
- **Zookeeper**：傳統模式下負責集群元數據管理、Leader 選舉與狀態監控。
- **KRaft**：新版 Kafka 引入的內置共識機制，移除了對 Zookeeper 的依賴，簡化了架構並提升了擴展性。

## Synergy: 協同工作流程
$\text{Producer} \xrightarrow{\text{Hash(Key)}} \text{Partition} \xrightarrow{\text{Sequential Write}} \text{Broker (Log File)} \xrightarrow{\text{Offset}} \text{Consumer Group} \rightarrow \text{Processing}$

這種設計將 **寫入的順序性** 與 **讀取的並行性** 完美結合，使得 Kafka 能在處理海量數據的同時，依然保持低延遲。