+++
date = '2025-02-05T10:30:00+08:00'
draft = false
title = '軟體介入解決硬體雜訊 (Software Intervention to Resolve Hardware Noise)'
description = '分享如何透過軟體介入解決 SSD 韌體產生的誤報風暴，平衡技術正確性與客戶體驗。'
categories = ["系統工程 (System Engineering)", "問題解決 (Problem Solving)"]
tags = ["SSD", "硬體監控 (Hardware Monitoring)", "務實工程 (Pragmatic Engineering)", "顧客至上 (Customer Obsession)"]
ShowToc = true
TocOpen = true
+++

## 簡介 (Introduction)

在系統工程 (system engineering) 中，當你的軟體必須跟底層的硬體、韌體、驅動程式互動時，即使軟體上的邏輯正確，仍可能被底層的雜訊所干擾。

## 問題：連鎖誤報 (The Problem: A Cascade of False Alarms)

我們一個 SSD 監控常駐程式 (daemon)，負責SSD的健康狀態給客戶。
有一天，一位客戶的 SSD 突然大規模觸發 SSD 耗損警報(wear-out metrics) ，
加上他們快要發布新產品了，所以事件上是有點壓力的

假設 bug 是我們造成的，我檢查程式碼，程式碼完全按照設計讀取作業系統訊號 (OS signals) 並如實回報，看起來一切正常。

但客戶的報告表示**訊號本身就是不穩定的 (flaky)**。

在我們的 daemon 底層，特定的 SSD 韌體 (firmware) 會在較低層級產生偽陽性 (false positives)。而我們的軟體僅僅忠實地將訊號回報，導致這種客戶的警報。

這造成了一個兩難：

- 實作方式是 **技術上正確的**，但它卻 **並非使用者預期的行為**，因為它產生的是雜訊。
- 硬體供應商目前也沒有接到相關回報，若需要進一步協助查明，曠日費時，肯定無法趕上客戶近期的發布。

## 解決方案：啟發式過濾器 (The Solution: A Heuristic Filter)


所以我實作了一個防抖動 (debouncing) 機制。

我們的軟體要求訊號必須在多次讀取中（或在一個短時間窗口內）持續存在。如果訊號是暫時性的，且在下一次檢查時消失，我們就將其視為雜訊。如果它持續存在，我們才視為真實情況。

* 取捨 (tradeoff) 
    在於延遲 (latency)。我們的 daemon 每隔幾秒檢查一次訊號，所以「確認」訊號的持續性可能會將警報延遲幾秒鐘

    對於 SSD 健康監控來說，並不是毫秒級的緊急事件，它是一種趨勢。在這種情境下，回報稍微延遲但可靠是可以接受的方案。 
    我也跟主管確認了，這是一個務實的修補程式 (pragmatic patch)，目的是為了幫助客戶如期發布，同時推動供應商進行後續修復計畫。

## 結果：解鎖與學習 (The Result: Unblocked and Learned)

修補程式順利部署。客戶如期發布了產品。

話雖如此，更深層的問題仍是一個謎。我們尚未確認生產過程中導致這不穩定行為的確切環節 (culprit stage)。我已請我們的磁碟團隊繼續追蹤並推動根本原因的分析 (root-cause diagnosis)。

對我來說，更大的教訓在於「軟體行為的正確性」在現實世界中的意義：

如果軟體的工作是幫助客戶自信地維運系統，那麼工作就不僅僅是傳遞訊號。我們必須將充滿雜訊的現實轉化為穩定、可解釋且可採取行動的資訊。

---


**AI寫作輔助說明**
本文部分結構整理與模型符號排版，使用生成式 AI 協助語句修訂與數學式呈現。核心觀點、論述架構與案例選擇皆為作者自行構思與負責。