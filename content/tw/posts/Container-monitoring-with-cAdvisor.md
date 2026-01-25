+++
date = '2025-10-11T16:43:38+08:00'
draft = false
title = '使用 cAdvisor 進行容器監控'
tags= ["Observibility"]
series= [""]
+++

# 簡介 (Introduction)

現今在微服務架構中，工程師會架設多個容器來提供服務，容器監控成為熱門議題。在本文中，我們使用 [cAdvisor](https://github.com/google/cadvisor) 來監控主機上所有運行的容器，並使用 Grafana 視覺化資料。

# 設定 (Setup)

本實驗的腳本
<https://github.com/nikeasyanzi/Prometheus-Grafana-Cadvisor-Docker>

以下是 docker compose 檔案中 cadvisor 的配置。

```yaml
cadvisor:
  container_name: cadvisor
  image: gcr.io/cadvisor/cadvisor:latest
  ports:
    - "8080:8080"
  privileged: true
  volumes:
    - "/:/rootfs:ro"
    - "/var/run:/var/run:rw"
    - "/var/lib/docker/:/var/lib/docker"
    - "/sys:/sys:ro"
    - "/dev/disk/:/dev/disk"
  devices:
    - "/dev/kmsg" #  for kernel message
```

所有容器啟動並運行後，
![](https://i.imgur.com/d072x3l.png)

我們透過連結 <http://localhost:8080/containers/> 檢查 cAdvisor。
我們可以看到所有運行中的容器清單

![](https://i.imgur.com/tLmdWTM.png)

另外，向下捲動頁面，我們可以看到每個容器運行時的 CPU 和記憶體使用情況。
![](https://i.imgur.com/nTzL1Do.png)

然後我們前往 Grafana，<http://localhost:3000>
我們新增一個儀表板來視覺化容器的資料。

- 用於 cadvisor：[https://grafana.com/grafana/dashboards/13946](https://grafana.com/grafana/dashboards/13946)
  ![](https://i.imgur.com/hoyOB6h.png)

為了實驗目的，我們使用 k6 壓力測試來產生流量，並觀察流量是否反映在我們的系統上。
![](https://i.imgur.com/IBNiAJE.png)

切換到儀表板並在底部，我們可以看到 RX/TX 流量進來。
![](https://i.imgur.com/4WkcDFI.png)

# 結論 (Conclusion)

我們成功建立了容器監控系統。包括 1. 設置 cAdvisor 來監控容器指標 2. 整合 cAdvisor 與 Grafana 進行視覺化

此外，我們使用 k6 產生負載來驗證系統能力。

# 參考資料 (Reference)

<https://medium.com/@jsantoine24/monitoring-docker-containers-with-cadvisor-ed21a9cfae95>
<https://blog.darkthread.net/blog/cadvisor-prometheus-grafana/>

<https://medium.com/@sohammohite/docker-container-monitoring-with-cadvisor-prometheus-and-grafana-using-docker-compose-b47ec78efbc>

<https://blog.devops.dev/monitoring-with-cadvisor-prometheus-and-grafana-on-docker-8fc5c4a2eae7>

<https://medium.com/@varunjain2108/monitoring-docker-containers-with-cadvisor-prometheus-and-grafana-d101b4dbbc84>
