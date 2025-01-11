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
- once the prometheus service is up, check with **public_ip:9090** in browser to access prometheus web console.

## Node_exporter setup in agents

- Once prometheus setup was completed, setup the node_exporter in agent servers.
- This node_exporter will exports all the server configurations to prometheus server to monitor the agents.
- You can get the node_exporter on "[Prometheus Official Download](https://prometheus.io/download/)".
- install the node_exporter in **/opt** directory on agent servers
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz 
```
- Extract the tar file
```bash
tar -zxvf node_exporter-1.8.2.linux-amd64.tar.gz 
```
- Rename the file if required
```bash
mv node_exporter-1.8.2.linux-amd64 node_exporter
```
- create systemctl service for node_exporter
```bash
vi /etc/systemd/system/node_exporter.service
```
- paste below content in it
```bash
[Unit]
Description=Prometheus Server


[Service]
ExecStart=/opt/node_exporter/node_exporter 


[Install]
WantedBy=multi-user.target
```
- Enable node_exporter service
```bash
systemctl enable node_exporter
```
- Start node_exporter service
```bash
systemctl start node_exporter
```
- check status of the service
```bash
systemctl status node_exporter
```
- check listener port is up or not
```bash
netstat -lntp
```
- After setup node_exporter you need to add the agent server details in prometheus.yml file on proemtheus server
- Go to below path and open prometheus.yml file in prometheus server
```bash
cd /opt/prometheus/
vi prometheus.yml
```
- In that file we can add agent details on service discovery(**scrape_configs:**) section in two ways
- one is manually by adding server ip address details as mentioned below:
```bash
- job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
- job_name: "Node-1"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["172.31.91.182:9100"]
        labels:
          -  name: 'Node-1'
- job_name: "Node-2"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["172.31.87.11:9100"]
        labels:
          -  name: 'Node-2'
```
