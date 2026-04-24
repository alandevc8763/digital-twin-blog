---
title: "Local Ai Infra"
date: 2026-04-24
categories: ["AI"]
tags: ["AI", "Ai-Engineering", "Local-Ai-Infra.Md"]
draft: false
---

# 構建私有 AI 研發環境：從 Proxmox GPU 穿透到本地 LLM 生產線
很多 AI 工程師的開發流程是這樣的：寫一個 Prompt $\rightarrow$ 調用 OpenAI API $\rightarrow$ 等待結果 $\rightarrow$ 調整 Prompt $\rightarrow$ 重複。
這種模式在原型階段很高效，但在工程實踐中存在三個致命缺陷：**隱私洩漏、Token 成本不可控、以及對第三方 API 的絕對依賴**。如果你在構建一個企業級的 AI 應用，將「推理能力」視為外部依賴，就如同將數據庫託管在一個你無法控制其延遲的第三方服務上。
對於 SRE 或 DevOps 工程師來說，真正的解決方案是構建一套 **「主權 AI (Sovereign AI)」** 基礎設施。
## 第一層：打破虛擬化的壁壘 (GPU Passthrough)
要在 HomeLab 中運行本地 LLM，首先要解決的是 GPU 的訪問問題。
很多人嘗試在 LXC 容器中部署 GPU，但如果你追求的是生產級的穩定性，**VM (虛擬機) + PCIe Passthrough** 是唯一的選擇。LXC 雖然輕量，但在處理 NVIDIA 驅動版本更新或內核模塊衝突時，經常會導致宿主機崩潰。
**SRE 的實踐路徑：**
1. **IOMMU 隔離**：在 Proxmox 的 GRUB 中開啟 `intel_iommu=on`，確保 GPU 被正確隔離到獨立的 IOMMU 組中。
2. **PCI Passthrough**：將 GPU 直接映射給 Ubuntu Server VM。這樣 VM 看到的不是「虛擬 GPU」，而是一個原生的 PCIe 設備。
3. **驅動對齊**：在 VM 內部安裝匹配的 CUDA Toolkit。
**避坑指南**：如果你遇到 `nvidia-smi` 在宿主機可用但在 VM 中失效，通常是 PCIe 電源狀態或 IOMMU 分 grouped 過大導致的。這時需要開啟 `pcie_acs_override` 來強制分割設備組。
## 第二層：推理引擎的選擇 (vLLM vs Ollama)
有了硬件訪問權後，接下來是選擇「推理引擎」。這裡的關鍵在於 **吞吐量 (Throughput)** 與 **延遲 (Latency)** 的權衡。
- **Ollama**：它是 AI 界的「Docker」，極其簡單。適合快速原型開發和單人使用。
- **vLLM**：如果你在構建一個需要支持多個併發請求的應用，vLLM 是工業標準。它引入了 **PagedAttention** 技術，將 KV Cache (鍵值快取) 像操作系統管理內存分頁一樣進行管理，極大地提升了顯存利用率。
**關於量化 (Quantization) 的權衡**：
你不需要 16-bit 的原版模型。透過 **GGUF** 或 **EXL2** 量化到 4-bit 或 8-bit，你可以用 24GB 的 VRAM 跑起原本需要 80GB 的模型，而精度損失在大多數應用場景中幾乎不可察覺。**本地 AI 的遊戲規則就是：用量化換取更大的模型參數，因為模型規模 $\rightarrow$ 智能水平 $\rightarrow$ 最終效果。**
## 第三層：從「聊天機器人」到「生產線」 (Local RAG)
本地 LLM 僅僅是「大腦」，要讓它有用，必須給它「記憶」與「工具」。
這就是將本地 AI 基礎設施與 **RAG (檢索增強生成)** 結合的時刻。在私有環境中，你可以構建一個完全閉環的數據流：
`本地文檔 (Markdown/PDF)` $\rightarrow$ `本地 Embedding 模型` $\rightarrow$ `本地向量庫 (Milvus/Qdrant)` $\rightarrow$ `本地 LLM (vLLM)`。
**這種架構的絕對優勢在於：**
- **零延遲**：數據在本地局域網流動，無需經過公網。
- **零成本**：除了電費，你不再需要為每一萬個 Token 付費。
- **絕對私有**：你的核心代碼庫或公司私密文檔永遠不會離開你的伺服器。
## 總結：構建 AI 時代的「數字主權」
構建本地 AI 基礎設施並非為了追求「極客感」，而是一種對 **計算控制權** 的奪回。
當你擁有從 Proxmox 底層、GPU 驅動、推理引擎到 RAG 應用鏈路的完整掌控力時，你就不再是一個單純的 API 調用者，而是一個真正的 **AI 系統工程師**。
**最好的 AI 應用，不是依賴於最強的雲端模型，而是依賴於一套可控、可演進且高度集成的本地基礎設施。**


---

## 🔗 Related Insights

- The Proxmox foundation required for this AI stack: Local Ai Infra Proxmox [TBD]
- Tuning the underlying CPU topology for LLM throughput: [Cpu Numa](../performance/cpu-numa.md)