# AzureVM-Grafana-Prometheus
Azure VM Grafana Prometheus 

Below is a **clean, production-ready guide** to set up **Grafana + Prometheus + Node Exporter on an Azure VM** (Ubuntu).
This is **trainer-friendly**, **lab-ready**, and suitable for **DevOps / Cloud monitoring demos**.

---

![Image](https://s3.amazonaws.com/a-us.storyblok.com/f/1022730/c982ad8d75/prometheus-grafana-architecture-diagram.png)

![Image](https://www.opsramp.com/wp-content/uploads/2022/07/node-img-1-1024x487-1.jpeg)

![Image](https://miro.medium.com/1%2A9_XeHLdBdRveDxVVBPf_kA.png)

![Image](https://volkovlabs.io/img/blog/2023-09-26-timeseries-dashboard/timeseries-prometheus.png)

---

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

1. **Azure VM**

   * OS: Ubuntu 20.04 / 22.04
   * Size: Standard_B1ms or higher

2. **NSG – Open Ports**

   | Service       | Port |
   | ------------- | ---- |
   | SSH           | 22   |
   | Node Exporter | 9100 |
   | Prometheus    | 9090 |
   | Grafana       | 3000 |

3. Login to VM:

```bash
ssh azureuser@<VM_PUBLIC_IP>
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
http://<VM_IP>:9100/metrics
```

---

## 🔹 Step 2: Install Prometheus

```bash
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.49.1/prometheus-2.49.1.linux-amd64.tar.gz
tar xvf prometheus-2.49.1.linux-amd64.tar.gz
sudo mv prometheus-2.49.1.linux-amd64 /etc/prometheus
```

### Configure Prometheus

```bash
sudo nano /etc/prometheus/prometheus.yml
```

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

### Create Prometheus Service

```bash
sudo nano /etc/systemd/system/prometheus.service
```

```ini
[Unit]
Description=Prometheus
After=network.target

[Service]
ExecStart=/etc/prometheus/prometheus \
  --config.file=/etc/prometheus/prometheus.yml

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

✅ Verify

```
http://<VM_IP>:9090
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
```

✅ Access

```
http://<VM_IP>:3000
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
