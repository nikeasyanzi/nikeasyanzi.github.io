+++
date = '2025-10-18T23:33:17+08:00'
draft = false
title = 'System Log Collection With Grafana Loki and Alloy'
tags= ["Observibility"]
series = [""]
+++

# Introduction

In this article, we demonstrate how to effectively collecting logs with [Alloy](https://github.com/grafana/alloy) and [Loki](https://github.com/grafana/loki).
With that, telemetry signals from applications, infrastructures are exported to on place for future telemetry.

In the lab exercise, we try to setup a Nginx service along with Loki, Alloy, and observe the system log on Grafana.

# Configuration deep dive

## Grafana Alloy configuration

A configuration file `config.alloy` is used, that contains logs to collect and where to forward them to.

## Loki Configuration

Grafana Loki requires a configuration file to define how it should run. Within the loki-fundamentals directory, you will find a file called _loki-config.yaml_.

## Grafana Loki Data source

This is used by Grafana to connect to Loki and query the logs. Grafana has multiple ways to define a data source;

- Direct: This is where you define the data source in the Grafana UI.
- Provisioning: This is where you define the data source in a configuration file and have Grafana automatically create the data source.
- API: This is where you use the Grafana API to create the data source.

In the following lab part, we are using the provisioning method. We have defined the data source in this portion of the docker-compose.yml file:

# Lab exercise

The full configuration and docker-compose file can be found [here](https://github.com/nikeasyanzi/loki-alloy-Grafana)
In the repo,
_config.alloy_ is the configuration file for alloy.
_loki-config.yam_ is the configuration file for loki.
grafana-dashboard.json
grafana-datasources.yml defining the datasource for grafana
grafana-default.yaml
grafana-data is the folder for grafana database

## Starting the service

```
docker-compose -f ./docker-compose.yml up -d
```

![](https://i.imgur.com/0U9B96d.png)

Next, we check the service status.

1. nginx
   <http://10.122.168.81:8082/stub_status>
2. loki
    [http://localhost:3100/metrics](http://localhost:3100/metrics)
3. alloy
   <http://localhost:12345/>

When checking the alloy status, we see the loki components are up and running.
![](https://i.imgur.com/vL5XLtX.png)

We can also navigate the Alloy UI at <http://localhost:12345/graph> to see the configuration file `config.alloy`visually. The configuration file defines logs to collect and where to forward them to.

- discovery.docker: This component queries the metadata of the Docker environment via the Docker socket and discovers new containers, as well as providing metadata about the containers.

- discovery.relabel: This component converts a metadata (\_\_meta_docker_container_name) label into a Loki label (container).

- loki.source.docker: This component collects logs from the discovered containers and forwards them to the next component. It requests the metadata from the discovery.docker component and applies the relabeling rules from the discovery.relabel component.

- loki.process: This component provides stages for log transformation and extraction. In this case it adds a static label env=production to all logs.

- loki.write: This component writes the logs to Loki. It forwards the logs to the Loki endpoint <http://loki:3100/loki/api/v1/push>.

![](https://i.imgur.com/SR3QRQc.png)

4. Next, we configure Grafana to visualize the data collected by loki.
   ![](https://i.imgur.com/ebB1Tdi.png)

Click Save&test to check the connectivity.
![](https://i.imgur.com/CBBP1Pa.png)

## Check logs of our services

Once the service is up and running, we can navigate to <http://localhost:3000/drilldown>.
Select Logs. You should see the Grafana Logs Drilldown page.

We can see Alloy is able to tail the Nginx log, also other container's logs are also collected.
![](https://i.imgur.com/GQjy4oQ.png)

## Query logs

To manually query Loki to ask more advanced questions about the logs, we use Grafana Explore.
Open a browser and navigate to <http://localhost:3000> to open Grafana.
From the Grafana main menu, click the Explore icon (1) to open the Explore tab.
Click **Code** to work in Code mode and type **{container="nginx"}**
![](https://i.imgur.com/cj7PM4O.png)

Also here , we keep restart the Nginx service every 30s.

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

所以其實寫到system log 即可

# Conclusion

This note demonstrated how to set up a log collection system using:
**Grafana Alloy**, **Loki** and **Grafana**.

1. how to collect logs using **Grafana Alloy** and send to **Loki**
2. how the collected logs been configured in **Grafana**.

To sum up, the solution provides a way to redirect the log used to be generated at local to remote Grafana serve. That provides a unified interface.

# Reference

<https://medium.com/@venkat65534/full-stack-observability-with-grafana-prometheus-loki-tempo-and-opentelemetry-90839113d17d>

<https://github.com/grafana/loki-fundamentals>

<https://grafana.com/docs/loki/latest/get-started/quick-start/tutorial/>

<https://github.com/grafana/docker-monitor-workshop>

<https://ithelp.ithome.com.tw/articles/10335935>

<https://medium.com/@fawenyo/python-%E7%9B%A3%E6%8E%A7-prometheus-loki-grafana-25ead4bbb681>

<https://medium.com/@derekjan1240/grafana-loki-prometheus-with-docker-compose-75d431bd07e2>
