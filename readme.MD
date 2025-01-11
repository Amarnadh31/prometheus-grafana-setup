# Prometheus-Grafana Setup

- **Prometheus Setup**
- **Grafana Setup**

# if you want to setup prometheus-Grafana monitoring tool, follow below steps

## Prometheus Setup:

* For explaining purpose i have taken 1 master and 2 agent instances
* install the prometheus from offical page for the latest version "[Prometheus Official Download](https://prometheus.io/download/)". Below mentioned i'm used version for the setup

- switch to **root** user and go to **/opt** directory
- run below command to download the prometheus
```bash
wget https://github.com/prometheus/prometheus/releases/download/v3.1.0/prometheus-3.1.0.linux-amd64.tar.gz
```
- Extract the downloaded tar file
```bash
tar -zxvf prometheus-3.1.0.linux-amd64.tar.gz
```
- Rename the file name for better practices
```bash
mv prometheus-3.1.0.linux-amd64 prometheus
```
- create systemctl service for prometheus
```bash
vi /etc/systemd/system/prometheus.service
```
- paste below content in it
```bash
[Unit]
Description=Prometheus Server


[Service]
ExecStart=/opt/prometheus/prometheus --config.file=/opt/prometheus/prometheus.yml 


[Install]
WantedBy=multi-user.target
```
- Enable prometheus service
```bash
systemctl enable prometheus
```
- Start prometheus service
```bash
systemctl start prometheus
```
- check status of the service
```bash
systemctl status prometheus
```
- check listener port is up or not
```bash
netstat -lntp
```



