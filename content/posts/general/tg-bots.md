---
title: "Tg Bots"
date: 2026-04-24
categories: ["General"]
tags: ["General", "Automation", "Tg-Bots.Md"]
draft: false
---

# Telegram 憑證監控機器人：代碼優化實踐

## Taxonomy
- **Domain**: Automation / Security Monitoring
- **Signal**: 🥉 Bronze (Engineering Improvement)

## Core Insight
自動化工具的生命週期在於 **「可維護性」**。將硬編碼 (Hard-coded) 的參數轉移至配置文件 (YAML)，並通過指令化界面 (Command Interface) 暴露內部狀態，能將一個單純的「腳本」提升為一個「可運行的服務」。

## Implementation Details

### 🛠️ 優化核心：配置外部化
將 API Token、資料庫連接字串等敏感資訊從 Python 代碼移至 `config.yaml`。
**優點**：無需修改代碼即可切換環境（Dev $\rightarrow$ Prod），且能避免密鑰洩漏至 Git 倉庫。

### 🤖 功能擴展：狀態可視化
新增兩個核心指令提升可操作性：
- `/show_cert_info`：直接從 MongoDB 讀取並格式化輸出目標域名的證書詳情（Subject, Issuer, Expiry）。
- `/help`：提供自助式指令指南，降低用戶進入門檻。

### ⚙️ 技術棧
- **Language**: Python 3.9+
- **Storage**: MongoDB (儲存憑證狀態與監控清單)
- **Interface**: Telegram Bot API

## Synergy
- **Observability**：此模式可延伸至所有基礎設施監控工具，將「被動告警」轉化為「主動查詢」。
- **Agentic Workflow**：此機器人可作為 AI Agent 的一個「感知端 (Sensing Layer)」，讓 Agent 能夠監控憑證狀態並自動觸發更新流程。