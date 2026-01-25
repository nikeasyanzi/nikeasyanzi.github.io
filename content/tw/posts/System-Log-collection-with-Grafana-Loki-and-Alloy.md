+++
date = '2025-10-18T23:33:17+08:00'
draft = false
title = '使用 Grafana Loki 和 Alloy 收集系統日誌'
tags= ["Observibility"]
series = [""]
+++

# 簡介 (Introduction)

在本文中，我們將展示如何使用 [Alloy](https://github.com/grafana/alloy) 和 [Loki](https://github.com/grafana/loki) 有效地收集日誌。
透過這種方式，來自應用程式和基礎設施的遙測訊號會被匯出到統一的位置，以便未來進行遙測分析。

在實驗練習中，我們將嘗試設置 Nginx 服務，搭配 Loki、Alloy，並在 Grafana 上觀察系統日誌。

# 配置深入探討 (Configuration deep dive)

## Grafana Alloy 配置

我們使用配置檔案 `config.alloy`，其中包含要收集的日誌以及要轉發到哪裡。

## Loki 配置

Grafana Loki 需要一個配置檔案來定義其運行方式。在 loki-fundamentals 目錄中，你會找到一個名為 _loki-config.yaml_ 的檔案。

## Grafana Loki 資料來源

Grafana 使用此配置來連接 Loki 並查詢日誌。Grafana 有多種方式定義資料來源：

- Direct（直接）：在 Grafana UI 中定義資料來源。
- Provisioning（佈建）：在配置檔案中定義資料來源，讓 Grafana 自動建立資料來源。
- API：使用 Grafana API 建立資料來源。

在接下來的實驗部分，我們使用佈建方法。我們已在 docker-compose.yml 檔案的這部分定義了資料來源：

# 實驗練習 (Lab exercise)

完整的配置和 docker-compose 檔案可以在[這裡](https://github.com/nikeasyanzi/loki-alloy-Grafana)找到。
在這個儲存庫中：
_config.alloy_ 是 Alloy 的配置檔案。
_loki-config.yam_ 是 Loki 的配置檔案。
grafana-dashboard.json
grafana-datasources.yml 定義了 Grafana 的資料來源
grafana-default.yaml
grafana-data 是 Grafana 資料庫的資料夾

## 啟動服務 (Starting the service)

```
docker-compose -f ./docker-compose.yml up -d
```

![](https://i.imgur.com/0U9B96d.png)

接下來，我們檢查服務狀態。

1. nginx
   <http://10.122.168.81:8082/stub_status>
2. loki
    [http://localhost:3100/metrics](http://localhost:3100/metrics)
3. alloy
   <http://localhost:12345/>

檢查 Alloy 狀態時，我們看到 Loki 元件已啟動並正在運行。
![](https://i.imgur.com/vL5XLtX.png)

我們也可以瀏覽 Alloy UI，網址為 <http://localhost:12345/graph>，以視覺化方式查看配置檔案 `config.alloy`。該配置檔案定義了要收集的日誌以及要轉發到哪裡。

- discovery.docker：此元件透過 Docker socket 查詢 Docker 環境的中繼資料，並發現新容器，同時提供容器的中繼資料。

- discovery.relabel：此元件將中繼資料（\_\_meta_docker_container_name）標籤轉換為 Loki 標籤（container）。

- loki.source.docker：此元件從發現的容器中收集日誌，並將其轉發到下一個元件。它從 discovery.docker 元件請求中繼資料，並套用來自 discovery.relabel 元件的重新標記規則。

- loki.process：此元件提供日誌轉換和擷取的階段。在這種情況下，它會為所有日誌新增一個靜態標籤 env=production。

- loki.write：此元件將日誌寫入 Loki。它將日誌轉發到 Loki 端點 <http://loki:3100/loki/api/v1/push>。

![](https://i.imgur.com/SR3QRQc.png)

4. 接下來，我們配置 Grafana 來視覺化 Loki 收集的資料。
   ![](https://i.imgur.com/ebB1Tdi.png)

點擊 Save&test 檢查連線狀態。
![](https://i.imgur.com/CBBP1Pa.png)

## 檢查服務日誌 (Check logs of our services)

服務啟動並運行後，我們可以瀏覽 <http://localhost:3000/drilldown>。
選擇 Logs。你應該會看到 Grafana Logs Drilldown 頁面。

我們可以看到 Alloy 能夠追蹤 Nginx 日誌，其他容器的日誌也被收集了。
![](https://i.imgur.com/GQjy4oQ.png)

## 查詢日誌 (Query logs)

要手動查詢 Loki 以詢問更進階的日誌問題，我們使用 Grafana Explore。
開啟瀏覽器並瀏覽 <http://localhost:3000> 以開啟 Grafana。
從 Grafana 主選單中，點擊 Explore 圖示 (1) 以開啟 Explore 分頁。
點擊 **Code** 以在程式碼模式下工作，並輸入 **{container="nginx"}**
![](https://i.imgur.com/cj7PM4O.png)

另外，我們每 30 秒持續重啟 Nginx 服務。

```bash
#!/bin/bash
for i in {1..6}; do
  echo "This is loop number $i"
  docker restart nginx
  sleep 30
done
```

# How to add my application log to Grafana

如何自訂application log 呢？
從官網給的[greenhouse 範例](https://github.com/grafana/loki-fundamentals/tree/getting-started/greenhouse), 在in main_app.py, 我們看到

```python=
logging.basicConfig(
    format='ts=%(asctime)s,%(msecs)03d level=%(levelname)s line=%(lineno)d msg="%(message)s"',
    datefmt='%Y-%m-%d %H:%M:%S'
)

log = logging.getLogger('werkzeug')
log.setLevel(logging.INFO)
```

所以其實寫到 system log 即可

# 結論 (Conclusion)

本文展示了如何使用以下工具建立日誌收集系統：
**Grafana Alloy**、**Loki** 和 **Grafana**。

1. 如何使用 **Grafana Alloy** 收集日誌並發送到 **Loki**
2. 如何在 **Grafana** 中配置收集到的日誌。

總而言之，此解決方案提供了一種方法，可以將原本在本地產生的日誌重新導向到遠端 Grafana 伺服器。這提供了統一的介面。

# 參考資料 (Reference)

<https://medium.com/@venkat65534/full-stack-observability-with-grafana-prometheus-loki-tempo-and-opentelemetry-90839113d17d>

<https://github.com/grafana/loki-fundamentals>

<https://grafana.com/docs/loki/latest/get-started/quick-start/tutorial/>

<https://github.com/grafana/docker-monitor-workshop>

<https://ithelp.ithome.com.tw/articles/10335935>

<https://medium.com/@fawenyo/python-%E7%9B%A3%E6%8E%A7-prometheus-loki-grafana-25ead4bbb681>

<https://medium.com/@derekjan1240/grafana-loki-prometheus-with-docker-compose-75d431bd07e2>
