# Docker Prometheus Grafana Monitoring Guide

## Introduction
In today's tech landscape, monitoring infrastructure health and performance is essential for maintaining system reliability and efficiency. Prometheus and Grafana have emerged as popular tools for monitoring and visualizing metrics from various sources. In this guide, we'll walk through setting up a centralized Prometheus server and Grafana dashboard to monitor multiple servers using Docker containers. This guide focuses on using Docker individually for a beginner-friendly setup. You can create a compose file using the same commands and configurations as below.

## Prerequisites
Ensure you have the following ports open to the internet: 9091, 9100, and 9090.

## Setting Up Target Servers
First, deploy the necessary containers on your target servers using Docker.

### cAdvisor Container
Run the following command on each target server to deploy cAdvisor, which collects container metrics:
```bash
docker run -d -name=cadvisor -restart=always -p 9091:8080 -v /:/rootfs:ro -v /var/run:/var/run:rw -v /sys:/sys:ro -v /var/lib/docker/:/var/lib/docker:ro google/cadvisor:latest
```

I am using 9091 port here because most of the time 8080 is already taken by other apps. You can customize listening port.

### Node Exporter Container
Deploy the Node Exporter container to collect system metrics on each target server:
```bash
docker run -dit -p 9100:9100 -restart=always -name node-exporter prom/node-exporter
```

## Setting Up Central Prometheus Server
Next, set up a centralized Prometheus server to collect and store metrics from the target servers.

### Prometheus Container
Run the following command on your central server to deploy Prometheus:
```bash
docker run -dit -name prometheus -restart=always -p 9090:9090 -v /home/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

### Grafana Container
Setup Grafana on the central server as well. Deploy Grafana for visualizing metrics:
```bash
docker run -dit -name=grafana -restart=always -p 3000:3000 grafana/grafana
```

## Configuration
Edit the Prometheus configuration file (prometheus.yml) to specify the targets for scraping metrics. In my case, I stored it on the `/home/` directory; you can change it to any other directory using `-v /your/vm/path/file:/inside/docker/path/file`.

```yaml
# Prometheus configuration
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node_export"
    static_configs:
      - targets: ["target-server-ip:9100", "target-server-ip:9100", "target-server-ip:9100"]
  - job_name: 'cadvisor'
    scrape_interval: 10s
    metrics_path: '/metrics'
    static_configs:
      - targets: ['target-server-ip:9091', 'target-server-ip:9091', 'target-server-ip:9091']
```

## Conclusion
By following these steps, you can set up a centralized Prometheus server and Grafana dashboard to monitor multiple servers efficiently. With detailed metrics and visualization, you can gain valuable insights into your infrastructure's health and performance, enabling proactive maintenance and optimization. Happy monitoring!
