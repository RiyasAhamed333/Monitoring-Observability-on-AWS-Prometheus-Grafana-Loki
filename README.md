# 📊 Monitoring & Observability on AWS – Prometheus, Grafana, Loki

## 📌 Project Overview

This project implements a **production-style monitoring and observability stack on AWS** using:

- **Prometheus** for metrics collection  
- **Node Exporter & cAdvisor** for system and container metrics  
- **Grafana** for dashboards and visualizations  
- **Loki + Promtail** for log aggregation and querying  
- **Amazon CloudWatch** as an additional metrics source for EC2  

The architecture is built with **one central monitoring server (master EC2)** and **two application nodes (slave EC2s)**.  
Prometheus scrapes metrics from both app nodes, while Promtail ships logs from all nodes to a centralized Loki instance.

---

## 🏗️ High-Level Architecture

- **Monitoring Master EC2**
  - Prometheus
  - Grafana
  - Loki
  - Promtail (for master logs)

- **App/Slave EC2s (2×)**
  - Node Exporter (host metrics)
  - cAdvisor (container metrics)
  - Promtail (system + container logs)

Grafana visualizes:

- Host & container metrics from Prometheus  
- Logs from Loki via Promtail  
- EC2 metrics directly from **CloudWatch**  

---

## 📂 Project Structure

```text
aws-monitoring-observability/
│
├── master/
│   ├── docker-compose.yml          # Prometheus, Grafana, Loki, Promtail
│   ├── prometheus/
│   │   ├── prometheus.yml          # Scrape configs for master + app nodes
│   │   └── alert-rules.yml         # CPU, memory, container alerts
│   ├── loki/
│   │   └── loki-config.yml         # Single-node Loki configuration
│   └── promtail/
│       └── promtail-config.yml     # Collect master system logs
│
├── slave/
│   ├── docker-compose.yml          # Node Exporter, cAdvisor, Promtail
│   └── promtail/
│       └── promtail-config.yml     # Sends logs to Loki on master
└── README.md
```

🔄 Metrics & Logs Flow

| Flow | Component     | Description                                               |
| ---- | ------------- | --------------------------------------------------------- |
| 🖥️  | Node Exporter | Exposes CPU, memory, disk, network metrics from each EC2  |
| 📦   | cAdvisor      | Exposes container-level metrics from Docker on app nodes  |
| 📡   | Prometheus    | Scrapes Node Exporter & cAdvisor metrics from both slaves |
| 📊   | Grafana       | Visualizes Prometheus metrics and CloudWatch EC2 metrics  |
| 📜   | Promtail      | Tails system & container logs on each node                |
| 🗄️  | Loki          | Stores and indexes logs; queried from Grafana             |

🚀 Deployment Steps (High Level)
1️⃣ Create EC2 Instances

1 × Monitoring Master EC2

2 × App/Slave EC2s

All inside the same VPC and security group (or peered groups)

Open ports:

22 from your IP (SSH)

3000 (Grafana), 9090 (Prometheus) from your IP

3100 (Loki), 9100 (Node Exporter), 8080 (cAdvisor), 9080 (Promtail) inside VPC

2️⃣ Install Docker & Git

On each EC2:

sudo yum update -y     # or sudo apt update -y
sudo yum install -y docker git
sudo service docker start
sudo usermod -aG docker ec2-user  # or ubuntu


Log out and back in so Docker group is applied.

3️⃣ Clone Repo

On master:
```
git clone https://github.com/<your-username>/aws-monitoring-observability.git
cd aws-monitoring-observability/master
# Edit prometheus/prometheus.yml with private IPs of slave nodes
docker compose up -d
```

On each slave node:
```
git clone https://github.com/<your-username>/aws-monitoring-observability.git
cd aws-monitoring-observability/slave
# Edit promtail/promtail-config.yml:
#  - MONITORING_MASTER_PRIVATE_IP
#  - host: node-1 / node-2
docker compose up -d
```
🎛️ Grafana Setup

Open Grafana: http://<MonitoringMasterPublicIP>:3000
Default: admin / admin

Add Prometheus as a data source → http://prometheus:9090

Add Loki as a data source → http://loki:3100

Add CloudWatch as a data source:

Use IAM role or Access key/Secret key

Select region and EC2 namespace

Build dashboards:

System metrics (Node Exporter)

Container metrics (cAdvisor)

EC2 metrics (CloudWatch)

Log panels (Loki queries)

You can export your dashboards to JSON and store them later in a grafana/dashboards/ folder.

🚨 Alerts

Prometheus alert rules in:

master/prometheus/alert-rules.yml


Current sample alerts:

HighCPUUsage – CPU > 80% for 2 minutes

HighMemoryUsage – Memory usage > 80% for 3 minutes

Placeholder for container down alerts (customizable)

Integration with email/Slack can be added via Alertmanager if desired.

🏅 Skills Demonstrated

Prometheus metrics collection & alerting

Grafana dashboards (metrics + logs + CloudWatch)

Loki & Promtail log aggregation

Node Exporter & cAdvisor monitoring

Multi-node architecture on AWS (1 master, 2 slaves)

Docker Compose–based observability stack

Practical Monitoring & Observability for DevOps / SRE roles
