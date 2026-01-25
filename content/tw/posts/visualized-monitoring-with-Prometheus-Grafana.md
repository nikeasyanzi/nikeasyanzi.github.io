+++
date = '2025-09-05T16:43:38+08:00'
draft = false
title = '使用 Prometheus 和 Grafana 進行視覺化監控'
tags= ["Observibility"]
series= [""]
+++

# 簡介 (Introduction)

在本文中，我們將展示

1. 如何使用 Prometheus 來收集和監控我們的主機與 Nginx 服務。
2. 在 Grafana 中視覺化呈現收集到的指標

# 元件 (Components)

- **Prometheus** 收集並儲存時間序列資料形式的指標，亦即指標資訊會與記錄時的時間戳記一起儲存，以及稱為標籤的選用鍵值對。
- **Prometheus node exporter** 公開各種與硬體和核心相關的指標。
- **Grafana** 一種圖形化處理軟體，可支援不同型式的來源資料數據，並以豐富的圖形來呈現相關數據

# 配置 (Configuration)

## 更新 docker-compose

```yaml
services:
  nginx:
    image: nginx:latest
    container_name: nginx
    volumes:
      - ./nginx/:/etc/nginx/conf.d
    ports:
      - 8082:8080

  nginx-prometheus-exporter:
    image: nginx/nginx-prometheus-exporter:1.4
    container_name: nginx-prometheus-exporter
    command: -nginx.scrape-uri http://nginx:8080/stub_status
    ports:
      - 9113:9113
    depends_on:
      - nginx

  # System monitoring
  node-exporter:
    container_name: node-exporter
    image: prom/node-exporter
    ports:
      - 9100:9100

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yaml:/etc/prometheus/prometheus.yaml
      - ./prometheus_data:/prometheus
    command:
      - "--config.file=/etc/prometheus/prometheus.yaml"
    ports:
      - "9090:9090"
    depends_on:
      - node-exporter
      - nginx-prometheus-exporter

  #handles rendering panels & dashboards to PNGs
  renderer:
    image: grafana/grafana-image-renderer:3.4.2
    environment:
      BROWSER_TZ: Asia/Taipei
    ports:
      - "8081:8081"

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    volumes:
      - ./grafana_data:/var/lib/grafana
    environment:
      GF_SECURITY_ADMIN_PASSWORD: pass
      GF_RENDERING_SERVER_URL: http://renderer:8081/render
      GF_RENDERING_CALLBACK_URL: http://grafana:3000/
      GF_LOG_FILTERS: rendering:debug
    depends_on:
      - prometheus
    ports:
      - "3000:3000"
```

## 更新 prometheus.yaml

配置 Prometheus 抓取間隔

```yaml
global:
  scrape_interval: 5s # Server 抓取頻率
  external_labels:
    monitor: "my-monitor"
scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_exporter"
    static_configs:
      - targets: ["node-exporter:9100"]

  - job_name: "nginx_exporter"
    static_configs:
      - targets: ["nginx-prometheus-exporter:9113"]
```

# 啟動服務 (Start services)

現在，我們使用 docker-compose 來啟動服務。

1. 執行 docker-compose.yaml

```
docker-compose -f ./docker-compose.yaml up -d
```

2. 檢查服務狀態
   ![](https://i.imgur.com/IXdp3Vx.png)

# 服務檢查與疑難排解 (Service check and troubleshooting)

接下來，我們檢查服務狀態並確保指標已在瀏覽器上匯出。

1. Nginx 服務狀態 <http://localhost:8082/>
2. 檢查 Nginx 匯出的指標 <http://localhost:8082/stub_status>
    ![](https://i.imgur.com/WKbUrfu.png)
    Nginx 狀態資訊 \* Active connections: 2
    目前有 2 個活動連線。

        * server accepts handled requests: 2 2 2
         - **2**：已接受的連線數。 
         - **2**：已成功處理的連線數。 
         - **2**：總請求數（可能大於連線數，因為每個連線可以處理多個請求）。 

        * Reading: 0 Writing: 1 Waiting: 1
         - Reading: 0：正在讀取請求的連線數。 
         - Writing: 1：正在傳送響應的連線數。 
         - Waiting: 1：空閒等待新請求的連線數（使用 HTTP Keep-Alive）。
             P.S.當前伺服器負載較輕，共有 2 個連線，其中 1 個正在處理請求，1 個處於等待新請求的狀態。

3. 檢查 nginx-prometheus-exporter 收集的指標 <http://localhost:9113/metrics>
    ![](https://i.imgur.com/t4wzqGo.png)

4. 檢查 node exporter 收集的主機指標 <http://localhost:9100/metrics>
    ![](https://i.imgur.com/c0BlkdM.png)

5. 在 Prometheus 檢查端點狀態
    點擊 <http://localhost:9090/> 檢查 Prometheus 是否正在運行。
    我們可以輸入 **up** 來查詢當前存活的端點。

![](https://i.imgur.com/SGeiSt2.png)

我們可以輸入 **target** 來查詢當前存活的端點。
<http://localhost:9090/targets>
![](https://i.imgur.com/ex7oLXD.png)

現在，指標已被 Prometheus 收集並重新導向。

# 視覺化輸出 (Visualization output)

接下來，我們使用 Grafana 來建立視覺化資料。
前往 <http://localhost:3000/>
![](https://i.imgur.com/UNLL3wI.png)

Select **Data Source**
![](https://i.imgur.com/5zYyB0d.png)

Click **Test** to test the connectivity between Prometheus and Grafana.
![](https://i.imgur.com/YfTSUOS.png)

Then, we go to **Dashboards** and select **New** -> **New dashboard**.
We see the following page and select **import dashboard** to create new dashboard  
![](https://i.imgur.com/D1F3TQg.png)

[官網](https://grafana.com/grafana/dashboards/)上也有很多dashboard template. 這邊我使用 [Node exporter](https://grafana.com/grafana/dashboards/1860-node-exporter-full/) ID 1860 跟[Nginx exporter](https://grafana.com/grafana/dashboards/14900-nginx/) ID 14900.

Dashboards

- for nginx: [https://grafana.com/grafana/dashboards/12708](https://grafana.com/grafana/dashboards/12708)
- for node exporter : [https://grafana.com/grafana/dashboards/1860](https://grafana.com/grafana/dashboards/1860)

We select **load**
![](https://i.imgur.com/K35T2YB.png)

Also, select **Prometheus data source** and click **import**
![](https://i.imgur.com/fZJ3sEs.png)

Once the dashboard is created, we can see the list of dashboards.
![](https://i.imgur.com/O6cIwZU.png)

### Dashboard for Host

![](https://i.imgur.com/wduXcIe.png)

### Dashboard for Nginx

![](https://i.imgur.com/HII6wLv.png)

Now, we have visualized data for monitor.

To generate traffic, we can use [k6_script](https://www.markkulab.net/prometheus-monitoring-nginx-requests/)

```shell
cat k6_script.js| docker run --rm -i grafana/k6 run -
```

![](https://i.imgur.com/FvH1iaE.png)

![](https://i.imgur.com/xh7ktos.png)

# Conclusion

In this article, we demonstrate

1. Using Prometheus to collect and monitor our host machine and Nginx service.
2. The collected metric information is further visualized in Grafana

# Reference

<https://last9.hashnode.dev/how-to-download-and-run-node-exporter-using-docker#heading-step-2-run-the-node-exporter-container>

<https://github.com/880831ian/Prometheus-Grafana-Docker>

<https://mxulises.medium.com/simple-prometheus-setup-on-docker-compose-f702d5f98579>

<https://github.com/880831ian/Prometheus-Grafana-Docker/tree/master>
