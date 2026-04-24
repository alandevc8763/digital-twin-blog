# 從 Ingress 到 Gateway API：Kubernetes 流量管理的範式轉移

如果你在 K8s 中管理過流量，你一定被 Ingress 的「註釋地獄 (Annotation Hell)」折磨過。

為了實現一個簡單的金絲雀發佈 (Canary Release) 或路徑重寫，你得在 YAML 裡寫上一大堆像 `nginx.ingress.kubernetes.io/rewrite-target: /` 這樣的註釋。這不僅讓配置變得難以維護，更糟糕的是，這些註석是**供應商綁定**的 $ightarrow$ 如果你從 Nginx 換到 Istio，所有的配置全部得重寫。

這就是為什麼 Kubernetes 推出了 **Gateway API**。它不是對 Ingress 的小修小補，而是一次權責分離的範式轉移。

## 1. 角色分離：誰該負責什麼？

Ingress 的問題在於它試圖用一個資源解決所有問題。而 Gateway API 引入了**角色導向 (Role-oriented)** 的模型：

- **基礎設施提供商 (Infrastructure Provider)** $ightarrow$ 定義 `GatewayClass`：決定使用哪種硬體或軟體 (例如 AWS ALB 或 Istio)。
- **叢集運維人員 (Cluster Operator)** $ightarrow$ 定義 `Gateway`：實例化一個負載均衡器，決定它監聽哪個端口、哪個證書。
- **應用開發者 (App Developer)** $ightarrow$ 定定義 `HTTPRoute` / `TCPRoute`：決定流量如何分發到自己的 Service。

**這種分離意味著：** 開發者可以自由地管理自己的路由規則，而不需要擁有整個叢集或負載均衡器的管理權限，實現了真正的自服務。

## 2. 核心能力：從「漏水抽象」到「標準化」

Ingress 是一個「漏水抽象 (Leaky Abstraction)」，很多高級功能得靠廠商私有的註釋來實現。Gateway API 將這些能力直接寫入了標準 API 規範中：

- **權重流量切分 (Traffic Splitting)**：原生支持 `weight: 10%` $ightarrow$ `weight: 90%`，無需依賴 Nginx 插件。
- **標頭路由 (Header-based Routing)**：根據 HTTP Header (例如 `user-agent: mobile`) 輕鬆導流。
- **跨命名空間路由 (Cross-namespace Routing)**：允許一個 Gateway 跨越 Namespace 將流量導向不同的服務，極大簡化了多租戶架構。

## 3. SRE 視角：為什麼你應該遷移？

從運維角度看，Gateway API 帶來了兩個核心收益：

1. **GitOps 友好的結構**：開發者只需提交一個 `HTTPRoute` 資源，無需修改複雜的 Ingress 配置，降低了誤操作導致全站崩潰的風險。
2. **生態系統融合**：它與 **Istio** 和 **Linkerd** 等 Service Mesh 深度集成。你可以用一套 API 統一管理從邊緣 (Edge) 到內部 (Internal) 的所有流量路徑。

## 結語：擺脫註釋，擁抱標準

如果你還在為 `rewrite-target` 的正則表達式抓狂，是時候考慮 Gateway API 了。它將流量管理從「配置技巧」提升到了「架構設計」的高度。

---
**技術關鍵字：**
- Gateway API (K8s 流量 API)
- Role-oriented Model (角色導向模型)
- HTTPRoute (HTTP 路由)
- Traffic Splitting (流量切分)
- Canary Release (金絲雀發佈)
