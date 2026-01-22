# AzureVM | Grafana, Prometheus, Node-Exporter

Below is a **clean, production-ready guide** to set up **Grafana + Prometheus + Node Exporter on an Azure VM** (Ubuntu).
This is **trainer-friendly**, **lab-ready**, and suitable for **DevOps / Cloud monitoring demos**.

---

### 🚀 Why use **Grafana + Prometheus + Node Exporter** (in short)

* **📊 Grafana** – Visualize metrics with **dashboards & graphs**
* **📥 Prometheus** – **Collects, stores, and queries** time-series metrics
* **🖥️ Node Exporter** – Exposes **server/VM metrics** (CPU, memory, disk, network)

---

### 🔑 Combined Benefits

* **📈 Real-time system monitoring**
* **⚠️ Early alerting** on CPU, memory, disk, service issues
* **🔍 Deep visibility** into Linux server performance
* **🧩 Open-source & cloud-native**
* **☁️ Works on VMs, containers, Kubernetes**
* **🛠️ Easy integration with alerts (Slack, Email, PagerDuty)**

---

### ✅ Typical Use Cases

* Monitor **Linux servers / cloud VMs**
* Track **application & infra health**
* **Troubleshoot performance issues**
* Build **SRE / DevOps observability stack**

## 🧩 Architecture Overview

**Flow**

```
Azure VM
 └─ Node Exporter (9100)  →  Prometheus (9090)  →  Grafana (3000)
```

**Purpose**

* **Node Exporter** → Collects OS & VM metrics
* **Prometheus** → Scrapes & stores metrics
* **Grafana** → Visualizes metrics via dashboards

---

## 🛠️ Prerequisites (Azure)

1. **Azure VM | Grafana, Prometheus**

   * OS: Ubuntu 20.04 / 22.04 | OS Disk | 128GB
   * Size: Standard_B1ms or higher
   * NSG - SSH - 22, Node-Exporter - 9100, Prometheus - 9090, Grafana - 3000

2. **NSG – Open Ports**

   | Service       | Port |
   | ------------- | ---- |
   | SSH           | 22   |
   | Node Exporter | 9100 |
   | Prometheus    | 9090 |
   | Grafana       | 3000 |

3. Login to VM:

```bash
cd Downloads 
chmod 400 key.pem 
ssh -i key.pem atul@20.62.198.127
yes
sudo apt update -y 
```

---

## 🔹 Step 1: Install Node Exporter

```bash
sudo useradd --no-create-home node_exporter
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvf node_exporter-1.7.0.linux-amd64.tar.gz
sudo mv node_exporter-1.7.0.linux-amd64/node_exporter /usr/local/bin/
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

### Create Service

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=default.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

✅ Verify

```
http://20.62.198.127:9100/metrics
```

---

## 🔹 Step 2: Install Prometheus

```bash
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.49.1/prometheus-2.49.1.linux-amd64.tar.gz
tar xvf prometheus-2.49.1.linux-amd64.tar.gz
sudo mv prometheus-2.49.1.linux-amd64 /etc/prometheus
sudo apt install prometheus
sudo snap install prometheus
```
```
sudo useradd \
  --system \
  --no-create-home \
  --shell /bin/false \
  prometheus
```

### Configure Prometheus

```bash
sudo nano /etc/prometheus/prometheus.yml
```

```
# Global configuration
global:
  scrape_interval: 15s
  evaluation_interval: 15s

# Alertmanager configuration (optional)
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - "localhost:9093"

# Rule files (optional)
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# Scrape configurations
scrape_configs:

  # Prometheus itself
  - job_name: "prometheus"
    static_configs:
      - targets:
          - "localhost:9090"

  # Node Exporter
  - job_name: "node_exporter"
    static_configs:
      - targets:
          - "localhost:9100"
```

### Create Prometheus Service

```bash
sudo nano /etc/systemd/system/prometheus.service
```

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/etc/prometheus/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries

Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```
sudo mkdir -p /var/lib/prometheus
sudo mkdir -p /etc/prometheus/consoles
sudo mkdir -p /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl restart prometheus
sudo systemctl status prometheus
```

✅ Verify

```
http://20.62.198.127:9090
```

---

## 🔹 Step 3: Install Grafana

```bash
sudo apt-get update
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install grafana -y
```

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

✅ Access

```
http://20.62.198.127:3000
```

* Username: `admin`
* Password: `admin` (change on first login)

---

## 🔹 Step 4: Configure Grafana → Prometheus

1. Login to Grafana
2. **Connections → Data Sources → Add data source**
3. Select **Prometheus**
4. URL:

```
http://localhost:9090
```

5. **Save & Test**

---

## 📊 Step 5: Import Node Exporter Dashboard

* Go to **Dashboards → Import**
* Dashboard ID: **1860**
* Select Prometheus datasource
* Import

🎉 You now have **CPU, Memory, Disk, Network, Load, Uptime metrics**.

---

## 🔐 Azure Best Practices (Trainer Notes)

✔ Use **Private IP** for Prometheus ↔ Node Exporter
✔ Restrict ports using **NSG source IP**
✔ For production:

* Enable **TLS**
* Add **Azure Monitor / Log Analytics**
* Use **Managed Identity**
* Persist data with external disk

---

## 📌 Common Interview Questions (Bonus)

* Why Prometheus is **pull-based**?
* Difference between **Azure Monitor vs Prometheus**?
* How to monitor **multiple Azure VMs**?
* How to scale Prometheus?
* Pushgateway vs Node Exporter?

---

## 🚀 Next Enhancements (Optional Labs)

* Add **Alertmanager**
* Monitor **AKS nodes**
* Azure Load Balancer + Grafana
* Prometheus federation
* Grafana provisioning via YAML
* Dockerized setup

---

**Alerting + Logs practice guide** that fits perfectly with your **Grafana + Prometheus + Node Exporter on Azure VM** lab.

---

# 🚨 ALERTING PRACTICE (Prometheus + Grafana)

## 🎯 What You’ll Practice

* Writing **PromQL alert rules**
* Triggering **real alerts**
* Visualizing alerts in **Grafana**
* Understanding **Alertmanager flow**

---

## 🔹 Practice 1: High CPU Alert (Node Exporter)

### Create Alert Rules File

```bash
sudo mkdir -p /etc/prometheus/rules
sudo nano /etc/prometheus/rules/node-alerts.yml
```

```yaml
groups:
- name: node-alerts
  rules:
  - alert: HighCPUUsage
    expr: 100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 80
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "High CPU usage detected"
      description: "CPU usage > 80% for more than 1 minute"
```

### Update Prometheus Config

```yaml
rule_files:
  - "rules/*.yml"
```

```bash
sudo systemctl restart prometheus
```

---

## 🔥 Trigger the Alert (Hands-On)

```bash
sudo apt install stress -y
stress --cpu 2 --timeout 120
```

### Verify

* Prometheus → **Status → Rules**
* Prometheus → **Alerts**
* Grafana → **Alerting → Alert rules**

---

## 🔹 Practice 2: Disk Space Alert

```yaml
- alert: DiskSpaceLow
  expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100 < 15
  for: 2m
  labels:
    severity: critical
  annotations:
    summary: "Low Disk Space"
    description: "Disk space below 15%"
```

---

## 🔹 Practice 3: Instance Down Alert

```yaml
- alert: InstanceDown
  expr: up == 0
  for: 1m
  labels:
    severity: critical
```

🔥 Trigger by stopping node exporter:

```bash
sudo systemctl stop node_exporter
```

---

## 📊 Grafana Alerting (Unified Alerts)

### Practice

1. Open **Node Exporter Dashboard**
2. Select CPU panel → **Create alert rule**
3. Condition:

```
WHEN avg() OF query(A) IS ABOVE 80
```

4. Evaluation: every 1m
5. Save alert

✅ Alert visible in Grafana **Alerting UI**

---

# 📜 LOGGING PRACTICE (Loki + Promtail + Grafana)

## 🎯 What You’ll Practice

* Collecting **VM logs**
* Querying logs with **LogQL**
* Correlating **Metrics + Logs**
* Debug-style troubleshooting

---

## 🔹 Step 1: Install Loki

```bash
wget https://github.com/grafana/loki/releases/download/v2.9.4/loki-linux-amd64
chmod +x loki-linux-amd64
sudo mv loki-linux-amd64 /usr/local/bin/loki
```

```bash
sudo nano /etc/loki-config.yml
```

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

common:
  path_prefix: /tmp/loki
  storage:
    filesystem:
      chunks_directory: /tmp/loki/chunks
      rules_directory: /tmp/loki/rules
  replication_factor: 1

schema_config:
  configs:
    - from: 2023-01-01
      store: tsdb
      object_store: filesystem
      schema: v13
      index:
        prefix: index_
        period: 24h
```

```bash
loki -config.file=/etc/loki-config.yml &
```

---

## 🔹 Step 2: Install Promtail

```bash
wget https://github.com/grafana/loki/releases/download/v2.9.4/promtail-linux-amd64
chmod +x promtail-linux-amd64
sudo mv promtail-linux-amd64 /usr/local/bin/promtail
```

```bash
sudo nano /etc/promtail-config.yml
```

```yaml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
- job_name: syslog
  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      __path__: /var/log/*.log
```

```bash
promtail -config.file=/etc/promtail-config.yml &
```

---

## 🔹 Step 3: Add Loki in Grafana

* **Connections → Data Sources**
* Select **Loki**
* URL:

```
http://localhost:3100
```

---

## 🔍 Logging Practice Queries (LogQL)

### View All Logs

```logql
{job="varlogs"}
```

### SSH Logs

```logql
{filename="/var/log/auth.log"}
```

### Errors Only

```logql
{job="varlogs"} |= "error"
```

### CPU + Logs Correlation (Interview Favorite)

* Spike CPU in Grafana
* Switch to **Logs panel**
* Inspect logs at same timestamp

---

# 🧠 Interview Practice Scenarios

### 🔹 Alerting

* Difference: **Prometheus Alert vs Grafana Alert**
* What happens if Prometheus restarts?
* How do you avoid alert fatigue?
* Alert severity vs routing

### 🔹 Logging

* Loki vs ELK
* Why labels matter in logs
* Metrics vs Logs vs Traces
* Log retention strategy

---

## 🏁 Practice Checklist (For Students)

✅ CPU Alert
✅ Disk Alert
✅ Instance Down Alert
✅ Grafana UI Alert
✅ Loki Installed
✅ Promtail Scraping Logs
✅ LogQL Queries
✅ Metrics + Logs correlation

---

## 🚀 Next Advanced Labs (Optional)

* Alertmanager email / Slack
* Log-based alerts
* Azure VM Scale Set monitoring
* AKS Alerts + Logs
* Tempo (Tracing)
* Full **Observability Stack**

---

Here’s a **handy list of popular Grafana Dashboard IDs** you can directly import into **Grafana** using **Dashboard → Import → ID**.

---

## 🔥 Most-Used Grafana Dashboard IDs

### 🖥️ **Node / Linux Monitoring**

| Dashboard                | ID        | Use Case                                                |
| ------------------------ | --------- | ------------------------------------------------------- |
| Node Exporter Full       | **1860**  | Complete Linux server metrics (CPU, RAM, Disk, Network) |
| Node Exporter Quickstart | **11074** | Lightweight node overview                               |
| Linux Host Metrics       | **10180** | Simple Linux monitoring                                 |

---

### 📊 **Prometheus Monitoring**

| Dashboard            | ID        | Use Case                   |
| -------------------- | --------- | -------------------------- |
| Prometheus 2.0 Stats | **3662**  | Prometheus server health   |
| Prometheus Overview  | **6417**  | General Prometheus metrics |
| Prometheus TSDB      | **12239** | Storage & TSDB internals   |

---

### ☸️ **Kubernetes / AKS / EKS**

| Dashboard                     | ID        | Use Case              |
| ----------------------------- | --------- | --------------------- |
| Kubernetes Cluster Monitoring | **315**   | Cluster-wide view     |
| Kubernetes Nodes              | **1860**  | Node-level metrics    |
| Kubernetes Pod Metrics        | **747**   | Pod-level monitoring  |
| Kubernetes API Server         | **12006** | Control plane metrics |

---

### 🐳 **Docker & Containers**

| Dashboard         | ID        | Use Case                   |
| ----------------- | --------- | -------------------------- |
| Docker Monitoring | **893**   | Docker engine & containers |
| cAdvisor          | **14282** | Container resource usage   |

---

### ☁️ **Cloud / Azure / AWS**

| Dashboard          | ID        | Use Case                   |
| ------------------ | --------- | -------------------------- |
| Azure VM Metrics   | **10956** | Azure VM monitoring        |
| AWS EC2 Monitoring | **617**   | EC2 instance metrics       |
| AWS CloudWatch     | **15106** | CloudWatch-integrated view |

---

### 🗄️ **Databases**

| Dashboard           | ID       | Use Case           |
| ------------------- | -------- | ------------------ |
| MySQL Overview      | **7362** | MySQL performance  |
| PostgreSQL Database | **9628** | PostgreSQL metrics |
| Redis Dashboard     | **763**  | Redis monitoring   |

---

### 🚀 **Application / Web**

| Dashboard        | ID        | Use Case          |
| ---------------- | --------- | ----------------- |
| NGINX            | **12708** | NGINX metrics     |
| Apache           | **3894**  | Apache web server |
| JVM (Micrometer) | **4701**  | Java applications |

---

## 📥 How to Import (Quick Steps)

1. Open **Grafana UI**
2. ➜ **Dashboards → Import**
3. Paste **Dashboard ID**
4. Select **Prometheus** data source
5. Click **Import**

---



Below is **HIGH CPU alert + auto-action script**

---

## 🚨 High CPU Alert (Prometheus – Node Exporter)

### 📄 `high-cpu-alert.yml`

```yaml
groups:
- name: cpu-alerts
  rules:
  - alert: HighCPUUsage
    expr: (100 - avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100) > 80
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High CPU usage detected"
      description: "CPU usage on {{ $labels.instance }} is above 80% for 5 minutes."
```

### 🔧 Enable in **Prometheus**

```yaml
rule_files:
  - "high-cpu-alert.yml"
```

```bash
sudo systemctl restart prometheus
```

---

## 🧪 Verify CPU Metric

```promql
100 - avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[2m])) * 100
```

---

## 🛠️ High CPU Troubleshooting Script (Linux)

### 📄 `high_cpu_check.sh`

```bash
#!/bin/bash

echo "===== HIGH CPU DIAGNOSTIC REPORT ====="
echo "Date: $(date)"
echo

echo "Top CPU Consuming Processes:"
ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head -10
echo

echo "CPU Load Average:"
uptime
echo

echo "CPU Core Usage:"
mpstat -P ALL 1 1
echo

echo "Zombie Processes:"
ps aux | awk '{ if ($8=="Z") print $0 }'
echo

echo "====================================="
```

### ▶️ Run

```bash
chmod +x high_cpu_check.sh
./high_cpu_check.sh
```

---

## ⚡ Auto-Kill High CPU Process (Optional – Use Carefully)

```bash
#!/bin/bash

THRESHOLD=85

PID=$(ps -eo pid,%cpu --sort=-%cpu | awk 'NR==2 {print $1}')
CPU=$(ps -eo pid,%cpu --sort=-%cpu | awk 'NR==2 {print $2}')

if (( ${CPU%.*} > THRESHOLD )); then
  echo "Killing PID $PID using $CPU% CPU"
  kill -9 $PID
fi
```

---

## 📌 Real-World Usage Flow

1. **Node Exporter** → exposes CPU metrics
2. **Prometheus** → fires `HighCPUUsage` alert
3. **Alertmanager** → sends Slack/Email
4. **Script** → identifies / mitigates root cause

---

## 🧠 Interview One-Liner

> *High CPU alerts are triggered using PromQL over node_cpu_seconds_total idle time, combined with diagnostic scripts to identify rogue processes.*

---
