# AWS MSK：實作 IAM 角色型身份驗證

## Taxonomy
- **Domain**: Cloud Infrastructure / Message Queue
- **Signal**: 🥈 Silver (Operational Best Practice)

## Core Insight
在企業級環境中，捨棄 SASL/SCRAM 而採用 **IAM 角色型驗證** 是為了實現「零密鑰管理」與「精細權限控制」。透過 AWS IAM，我們可以將權限直接綁定到執行個體的 Role，徹底消除密鑰洩漏風險，並簡化跨服務（如 MSK Connect）的集成過程。

## Implementation Details

### 🎯 叢集部署要點
1. **叢集設定**：選擇自訂建立 $\rightarrow$ 規格最小化（開發環境） $\rightarrow$ 3 個可用區 (AZ) 以確保高可用。
2. **網路配置**：使用預設 VPC，確保每個 AZ 擁有獨立子網。
3. **安全設定**：
   - **驗證方式**：選定 `IAM role-based authentication`。
   - **加密**：使用 AWS 受管金鑰加密靜態資料。

### 🔐 IAM Policy 權限拆解
為了確保最小權限原則，Policy 應分為三個維度：
- **Cluster Level** (`Connect`, `AlterCluster`, `DescribeCluster`)：允許 Client 端與叢集建立通信。
- **Topic Level** (`*Topic*`, `WriteData`, `ReadData`)：控制訊息的讀寫權限。
- **Group Level**：管理消費者分組的權限。

### 🛠️ 實作坑洞
- **連接器兼容性**：部分第三方連接器不支援 SASL/SCRAM，必須切換至 IAM 驗證方可正常運作。
- **建立時間**：MSK 叢集建立約需 15 分鐘，建議在 CI/CD Pipeline 中設置適當的等待時間。

## Synergy
- **Infrastructure as Code (IaC)**：此流程可直接轉化為 Terraform 或 CloudFormation 模組。
- **Zero Trust Architecture**：IAM 驗證是實現雲端零信任架構的核心步驟。
