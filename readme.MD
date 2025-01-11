# Prometheus-Grafana Setup

"[Prometheus Setup]()"
"[Grafana Setup]()"

## If you want to setup prometheus-Grafana monitoring tool, follow below steps

### Prometheus Setup:

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
- Go to below path and open prometheus.yml file in **prometheus server**
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
- Another way is by adding IAM role to prometheus server and get ec2 instances service discovery prometheus by filtering with the tags as shown below
```bash
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "ec2-instances"

    ec2_sd_configs:
      - region: us-east-1
        port: 9100
        filters:
          - name: tag:Monitoring
            values:
              - yes
    relabel_configs:
      - source_labels:
          - __meta_ec2_tag_Name
        target_label: name
```
- Once the prometheus.yml file updated, Restart the prometheus service
```bash
systemctl restart prometheus
```
- After restart and check in prometheus monitoring console by any query you'll able to see metrics for agent servers as well

- Up to now everything is fine, Now if anything happen to server means how can we get notified. As of now we only configured prometheus tool and connected agents to it but if anthing happen on server level means we need something to notify us.
- Here comed alertmanager in place which will notify if anything happens on server level. However we need to write alert rules for it.

## AlertManager setup in master server

- You can download the alertmanager on "[Prometheus Official Download](https://prometheus.io/download/)".
- install the alertmanager in **/opt** directory on master servers where prometheus was installed
```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.28.0-rc.0/alertmanager-0.28.0-rc.0.linux-amd64.tar.gz
```
- Extract the tar file
```bash
tar -zxvf alertmanager-0.28.0-rc.0.linux-amd64.tar.gz 
```
- Rename the file if required
```bash
mv alertmanager-0.28.0-rc.0.linux-amd64 alertmanager
```
- create systemctl service for alertmanager
```bash
vi /etc/systemd/system/alertmanager.service
```
- paste below content in it
```bash
[Unit]
Description=alertmanager Server


[Service]
ExecStart=/opt/alertmanager/alertmanager --config.file=/opt/alertmanager/alertmanager.yml 


[Install]
WantedBy=multi-user.target
```
- Enable alertmanager service
```bash
systemctl enable alertmanager
```
- Start alertmanager service
```bash
systemctl start alertmanager
```
- check status of the service
```bash
systemctl status alertmanager
```
- check listener port is up or not
```bash
netstat -lntp
```
- After alertmanager is up check with **public_ip:9093** in browser to access alert manager web console.
- you can update the **alertmaster.yml** in **/opt/alertmanager** path with your necessary changes
```bash
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 1h
  receiver: 'ses'
receivers:
  - name: 'ses'
    email_configs:
      - smarthost: 'email-smtp.us-east-1.amazonaws.com:587'
        auth_username: 'your-username'
        auth_password: "your-password"
        from: 'your-from-address'
        to: 'your-to-address'
        headers:
          subject: 'Prometheus Mail Alerts'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: [ 'alertname', 'dev', 'instance']
```