# Prometheus-Grafana Setup

## if you want to setup prometheus-Grafana monitoring tool, follow below steps

* For explaining purpose i have taken 1 master and 2 agent instances
* install the prometheus from offical page for the latest version "[Prometheus Official Download](https://prometheus.io/download/)". Below mentioned i'm used version for the setup

- switch to **root** user and go to **/opt** directory
- run below command to download the prometheus
```bash
wget https://github.com/prometheus/prometheus/releases/download/v3.1.0/prometheus-3.1.0.linux-amd64.tar.gz
