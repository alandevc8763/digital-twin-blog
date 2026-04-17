# AWS-CLI：高效帳號切換實踐

## Taxonomy
- **Domain**: Cloud Operations / CLI Efficiency
- **Signal**: 🥉 Bronze (Productivity Hack)

## Core Insight
對於管理多個 AWS 環境 (Dev/Staging/Prod) 的 SRE 而言，頻繁的帳號切換是最高頻的痛點。透過 **環境變數隔離** 與 **Shell Function 封裝**，可以將切換成本從「手動修改配置」降低至「單次指令」。

## Implementation Details

### 🚀 三種切換方案對比

| 方案 | 操作方式 | 適用場景 | 優缺點 |
| :--- | :--- | :--- | :--- |
| **--profile 參數** | `aws s3 ls --profile lab` | 單次指令 | $\checkmark$ 最安全 $\text{X}$ 輸入冗長 |
| **環境變數** | `export AWS_PROFILE=lab` | 當前 Session | $\checkmark$ 快速 $\text{X}$ 容易忘記切換導致誤操作 |
| **Shell Function** | `change_aws_profile lab` | 全局快速切換 | $\checkmark$ 極速 $\text{X}$ 需配置 `.zshrc/.bashrc` |

### 🛠️ 最佳實踐：Shell Function 封裝
在 `.zshrc` 或 `.bashrc` 中添加以下代碼：

```bash
change_aws_profile() {
    export AWS_PROFILE=$1
    export AWS_DEFAULT_PROFILE=$1
    echo "🚀 AWS profile switched to: $1"
}
```
**執行流程**：`source ~/.zshrc` $\rightarrow$ `change_aws_profile production` $\rightarrow$ 驗證身份 $\rightarrow$ 開始操作。

## Synergy
- **DevOps Pipeline**：此邏輯可延伸至 Jenkins/GitLab CI 的 Pipeline 環境變數管理。
- **Security**：建議配合 `aws-vault` 使用，將密鑰存儲於系統金鑰庫而非明文 `.aws/credentials`。
