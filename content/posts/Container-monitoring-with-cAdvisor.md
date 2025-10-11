+++
date = '2025-10-11T16:43:38+08:00'
draft = false
title = 'Container Monitoring With cAdvisor'
tags= ["Observibility"]
series= [""]
+++

# Introduction

Nowadays, in micro service architecture, engineers would host multiple containers to provide services, container monitor become a hot topic. In this post, we use [cAdvisor](https://github.com/google/cadvisor) to monitor all containers running on the host and visualize the data with Grafana.

# Setup

Scripts for this experiment
<https://github.com/nikeasyanzi/Prometheus-Grafana-Cadvisor-Docker>

Here is the configuration for cadvisor in the docker compose file.

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

Once all containers are up and running,
![](https://i.imgur.com/d072x3l.png)

We check cAdvisor by the link <http://localhost:8080/containers/>.
We can see the list of all running containers

![](https://i.imgur.com/tLmdWTM.png)

Also, scrolling the page down, we can see the CPU and memory usages of each container runtimes.
![](https://i.imgur.com/nTzL1Do.png)

Then we goto grafana, <http://localhost:3000>
We add a dashboard for the data visualization of the containers.

- for cadvisor: [https://grafana.com/grafana/dashboards/13946](https://grafana.com/grafana/dashboards/13946)
  ![](https://i.imgur.com/hoyOB6h.png)

For experiment purpose, k6 stress is used to generate traffics and see if the traffic is reflected on our system.
![](https://i.imgur.com/IBNiAJE.png)

Switching to the dashboard and at the bottom, we can see the RX/TX traffic coming in.
![](https://i.imgur.com/4WkcDFI.png)

# Conclusion

We successfully setting up a system for container monitoring. That includes 1. set up cAdvisor to monitor container metrics 2. integrated cAdvisor with Grafana for visualization

Also, we generate loading with k6 to validate the system capabilities.

# Reference

<https://medium.com/@jsantoine24/monitoring-docker-containers-with-cadvisor-ed21a9cfae95>
<https://blog.darkthread.net/blog/cadvisor-prometheus-grafana/>

<https://medium.com/@sohammohite/docker-container-monitoring-with-cadvisor-prometheus-and-grafana-using-docker-compose-b47ec78efbc>

<https://blog.devops.dev/monitoring-with-cadvisor-prometheus-and-grafana-on-docker-8fc5c4a2eae7>

<https://medium.com/@varunjain2108/monitoring-docker-containers-with-cadvisor-prometheus-and-grafana-d101b4dbbc84>
