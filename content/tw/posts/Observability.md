+++
date = '2026-01-23T16:43:38+08:00'
draft = 'False'
title = '可觀測性 Observability'
categories = ["Observability"]
tags = ["Observability"]
series= [""]
+++

# 可觀測性 (Observability)

![](https://i.imgur.com/FEPLgvU.png)
[來源](https://developer.ibm.com/articles/observability-insights-and-automation/)

**可觀測性**是一種實踐方法，透過對系統進行埋點來收集數據，讓你能夠從外部了解系統的內部狀態。

在現代複雜的環境中，如微服務架構，我們需要分層的可觀測性來幫助我們了解從業務層到底層基礎設施的狀態。可觀測性讓你能夠透過對系統行為提出任意問題，來探索和除錯那些你從未見過的問題（「未知的未知」）。

上圖金字塔清楚描繪了一個常見的業務場景：從發現服務停止運作，到 IT 團隊需要深入調查事件的過程。

為了達成這個目標，應用程式必須進行埋點以發送**遙測數據 (Telemetry)**——關於其效能和行為的數據。這些數據通常被分為三個基礎支柱。

## 可觀測性的三大支柱

![](https://i.imgur.com/OU4NjDB.png)

[來源](https://developer.ibm.com/articles/observability-insights-and-automation/)

雖然指標、日誌和追蹤常被稱為「三大支柱」，但它們真正的威力在於相互關聯，讓你能夠在它們之間無縫切換來診斷問題。

### 1. 指標 (Metrics)：「發生了什麼」
指標是在一段時間內收集的聚合數值數據。它們非常適合用來快速了解系統的整體健康狀況以及設定警報。例如：CPU 使用率、API 回應時間

*   **它們能回答的問題：**「CPU 使用率是多少？」、「我的 API 錯誤率是多少？」、「我們每秒處理多少請求？」

*   **關鍵概念：**
    *   **維度 (Dimensionality)：** 當指標具有維度（也稱為標籤或標記）時會更加強大，維度是提供上下文的鍵值對。例如，不只是 `http_requests_total`，你可以有 `http_requests_total{method="POST", status="500"}`。

    *   **基數 (Cardinality)：** 這是指維度的唯一組合數量。**高基數**（例如使用唯一的 `userID` 作為維度）對於監控系統的儲存和查詢效率來說是個挑戰。隨著時間序列數量的增長，查詢變得更加消耗計算資源。此外，任何重大事件，例如涉及不可變基礎設施的程式碼發布，都會導致大量同時寫入資料庫的操作。

*   **典型工具：** **Prometheus** 是收集和儲存時間序列指標數據的領先開源工具。

### 2. 日誌 (Logs)：「為什麼發生」
日誌是在特定時間發生的離散事件的帶時間戳、不可變的記錄。日誌提供了關於應用程式或系統內部發生什麼的最詳細、最細緻的上下文。例如：系統錯誤訊息或異常事件

* **它們能回答的問題：**「為什麼這個特定請求失敗了？」、「確切的錯誤訊息是什麼？」、「導致這次崩潰的事件順序是什麼？」

*  **典型工具：** **Grafana Loki** 是一個日誌聚合系統，設計目標是成本效益高且易於操作，特別是與 Prometheus 等其他工具整合時。

### 3. 追蹤 (Traces)：「在哪裡發生」
追蹤代表單一請求在分散式系統中穿越所有不同服務的端到端旅程。Span 追蹤請求發出的特定操作，描繪出在該操作執行期間發生了什麼。

例如：在 iThome 使用 Facebook 的 SSO（單一登入），一個單純的登入「請求」，可能就橫跨了 iThome 的後端服務、Facebook 的 SSO 後端、iThome 的 Redis，這些衍生出來的請求，與最初的登入請求結合起來呈現的歷程就稱為 Trace

*   **它們能回答的問題：**「這個使用者請求的延遲在哪裡？」、「哪個下游服務造成了瓶頸？」、「這個 API 呼叫通過我們微服務的完整路徑是什麼？」
*   **典型工具：** **Grafana Tempo** 是一個分散式追蹤後端，可以從各種來源儲存和檢索追蹤。

* 追蹤中的每個工作單位稱為 **span**。
	1. **單一追蹤中有大量 span：**
    這表示一個請求穿越了許多不同的服務或執行了大量操作。雖然這本身不一定是問題，但具有大量 span 的追蹤可能意味著高度複雜的交互，或者如果一個簡單的請求涉及太多微服務，可能是設計效率不佳。這可能使追蹤的視覺化呈現更加密集，但追蹤的主要目的就是繪製出這些複雜的旅程。
    
	2. **持續時間很長的 span（即在該 span 中花費了「大量」時間）：**
    這是追蹤提供的直接且關鍵的洞察。如果某個特定 span 持續時間很長，這意味著該 span 代表的特定操作或服務需要很長時間才能完成。這正是追蹤設計用來幫助識別的：
    
    - 「這個使用者請求的延遲在哪裡？」
    - 「哪個下游服務造成了瓶頸？」

    因此，持續時間「高」的 span 會精確定位系統中的效能瓶頸，讓你能夠進一步調查該特定操作或服務，可能透過跳轉到該服務在該確切時間的日誌來找到根本原因。

### 透過視覺化整合一切
最終目標是將這些支柱結合使用。典型的工作流程可能是：
1.  從**指標**觸發**警報**（例如，延遲過高）。
2.  查看慢請求的**追蹤**，看看哪個特定服務是瓶頸。
3.  從該追蹤跳轉到該服務在該確切時間的**日誌**，找到根本原因的錯誤訊息。

**Grafana** 是建立儀表板的事實標準工具，用於在單一介面中視覺化和關聯來自所有三個支柱的數據。

## 使用 OpenTelemetry 實現可觀測性
手動埋點程式碼以產生所有這些數據可能很複雜。**OpenTelemetry** 是一個廠商中立的開源可觀測性框架。它提供了一套標準化的 API、函式庫和代理程式，用於從應用程式收集和匯出遙測數據（指標、日誌和追蹤），使你不會被鎖定在單一廠商的生態系統中。

## 應用可觀測性：衡量服務可靠性
![](https://i.imgur.com/VjkRy2U.png)

[SLO vs SLA vs SLI 的區別](https://www.solarwinds.com/blog/differentiating-between-slo-vs-sla-vs-sli)

一旦你擁有豐富的遙測數據，你就可以超越單純的錯誤修復，開始衡量對使用者和業務真正重要的事情。這是透過 SLI、SLO 和 SLA 框架來完成的。

#### 1. 服務級別指標 (SLIs)
SLI 是服務效能的直接、可量化的衡量標準。它是你從可觀測性工具獲得的原始數據。
*   **範例：** 成功 HTTP 請求的百分比，或 95% 請求的延遲。

#### 2. 服務級別目標 (SLOs)
SLO 是你為某個 SLI 在一段時間內設定的**內部目標**。這是你的工程團隊努力達成的目標。
*   **範例：** 在 30 天內，99.9% 的 API 請求將會成功。

#### 3. 服務級別協議 (SLAs)
SLA 是對客戶的正式承諾，通常是合約性的，定義了如果未達到 SLO 的後果（例如，經濟處罰或服務抵免）。SLA 通常比內部 SLO 寬鬆，以提供緩衝。
*   **範例：** 承諾每月 99.5% 的正常運行時間，如果違反則提供 10% 的帳單抵免。

這個框架將可觀測性平台的技術數據直接與業務和客戶滿意度目標連結起來。

# 可觀測性
[可觀測性](https://opentelemetry.io/docs/concepts/observability-primer/#what-is-observability)讓你能夠從外部了解系統，透過對該系統提問，而無需知道其內部運作方式。

# 參考資料
[Observability primer](https://opentelemetry.io/docs/concepts/observability-primer/)

[Observability at Twitter: technical overview, part I](https://blog.x.com/engineering/en_us/a/2016/observability-at-twitter-technical-overview-part-i)

[Observability at Twitter: technical overview, part II](https://blog.x.com/engineering/en_us/a/2016/observability-at-twitter-technical-overview-part-ii)
[Observability 的過去與現在]
https://ithelp.ithome.com.tw/m/articles/10319113

[Why Your Observability Strategy Needs High Cardinality Data](https://betterstack.com/community/guides/observability/high-cardinality-observability/)

[Understanding High Cardinality in Observability](https://www.observeinc.com/blog/understanding-high-cardinality-in-observability)
