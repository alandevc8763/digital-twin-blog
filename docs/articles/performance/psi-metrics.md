# 別再看 CPU 利用率了：用 PSI 衡量系統的「痛苦程度」
很多 SRE 在分析性能瓶頸時，習慣看 `top` 裡的 CPU% 或 `free` 裡的 Mem%。
但這裡有一個巨大的陷阱：**利用率 (Utilization) $
eq$ 飽和度 (Saturation)**。
一個 CPU 利用率 50% 的系統，依然可能是卡死的。為什麼？因為剩下的 50% 時間，CPU 可能都在等待 I/O 返回，或者在等待記憶體分頁。
## 什麼是 PSI (Pressure Stall Information)？
PSI 是 Linux 內核提供的一套指標，它不再告訴你「資源被用了多少」，而是告訴你 **「任務因為等待資源而停頓了多久」**。
它將壓力分為三個維度：**CPU、Memory、I/O**。
### 1. 「Some」與「Full」的區別
- **Some**：至少有一個任務在等待資源，但系統還能跑其他東西。這是一個 **「警告信號」**。
- **Full**：所有非空閒任務全部在等待資源。CPU 處於完全空轉狀態。這是一個 **「崩潰信號」**。
### 2. 實戰分析：讀懂 PSI 信號
- **CPU Pressure High**: 如果 `some` 很高但 `full` 很低 $
ightarrow$ 你有太多線程在爭搶核心，上下文切換開銷太大。
- **Memory Pressure High**: 如果 `full` 突然飆升 $
ightarrow$ 系統正在進行劇烈的直接回收 (Direct Reclaim) 或 Swap。這通常是 OOM (Out of Memory) 發生前的最後警告。
- **I/O Pressure High**: 如果 `full` 持續處於高位 $
ightarrow$ 你的儲存設備已經飽和，應用程序被完全阻塞在 I/O 路徑上。
## SRE 實踐：基於壓力的自動擴展 (Pressure-based Scaling)
傳統的 Auto-scaling 是基於 CPU 閾值（例如 70% 擴容）。這很遲鈍，因為 CPU% 是滯後指標。
**更先進的做法是監控 `/proc/pressure/memory`**：
一旦 `some` 壓力超過 10% 並持續 30 秒 $
ightarrow$ 立即觸發擴容或清理快取。
這是在系統真正崩潰之前，提前捕捉到「痛苦」的唯一方法。
## 總結：監控「痛苦」而非「忙碌」
利用率告訴你資源有多忙，而 PSI 告訴你應用有多痛苦。**高性能監控的目標，是將關注點從 Utilization 轉移到 Saturation。**


---

## 🔗 Related Insights

- Tracing the actual I/O stalls in the VFS layer: [Vfs Tracing](vfs-tracing.md)
- How saturation metrics redefine SRE stability: [Sre Stability Myth](../sre-stability-myth.md)
