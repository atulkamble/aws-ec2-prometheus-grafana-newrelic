Excellent — let’s build a clean, practical **cloud observability stack project** using **Prometheus**, **Grafana**, and **New Relic** on an **EC2 instance**. I’ll give you:

* ✅ Project overview
* ✅ EC2 setup steps
* ✅ Installation scripts
* ✅ Prometheus & Grafana configuration
* ✅ New Relic agent installation
* ✅ Testing instructions
* ✅ Complete documentation

---

## 📊 EC2 Observability Stack: Prometheus + Grafana + New Relic

---

## 📌 Project Overview

We’ll deploy on a **single Amazon Linux 2 EC2 instance**:

* **Prometheus** → scrape metrics
* **Grafana** → visualize metrics from Prometheus
* **New Relic Infrastructure agent** → monitor instance and send telemetry to New Relic dashboard

---

## 📁 Directory Structure

```
ec2-observability-project/
├── install_prometheus_grafana.sh
├── prometheus/
│   └── prometheus.yml
├── test_exporter/
│   └── node_exporter.service
├── install_newrelic.sh
└── README.md
```

---

## 📦 Step 1️⃣: Launch EC2 Instance

* **AMI:** Amazon Linux 2
* **Security Group:**

  * SSH → 22
  * Prometheus → 9090
  * Grafana → 3000
  * Node Exporter → 9100

---

## 📦 Step 2️⃣: Install Prometheus & Grafana

Create `install_prometheus_grafana.sh`

```bash
#!/bin/bash

# Update packages
sudo yum update -y

# Install Prometheus
sudo useradd --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
tar -xvzf prometheus-2.52.0.linux-amd64.tar.gz
sudo mv prometheus-2.52.0.linux-amd64 /etc/prometheus
sudo mkdir -p /data/prometheus
sudo chown -R prometheus:prometheus /data/prometheus /etc/prometheus

# Copy config
sudo cp /etc/prometheus/prometheus.yml /etc/prometheus/prometheus.yml.backup

# Create Prometheus systemd service
sudo bash -c 'cat > /etc/systemd/system/prometheus.service << EOF
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
ExecStart=/etc/prometheus/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/data/prometheus

[Install]
WantedBy=multi-user.target
EOF'

sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus

# Install Grafana
sudo yum install -y https://dl.grafana.com/oss/release/grafana-11.0.0-1.x86_64.rpm
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

---

## 📦 Step 3️⃣: Prometheus Config

Create `prometheus/prometheus.yml`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

---

## 📦 Step 4️⃣: Install Node Exporter

Create `test_exporter/node_exporter.service`

```bash
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=ec2-user
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Then:

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
tar -xvzf node_exporter-1.8.1.linux-amd64.tar.gz
sudo mv node_exporter-1.8.1.linux-amd64/node_exporter /usr/local/bin/
sudo cp test_exporter/node_exporter.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

---

## 📦 Step 5️⃣: Install New Relic Infrastructure Agent

Create `install_newrelic.sh`

```bash
#!/bin/bash

# Replace YOUR_LICENSE_KEY with actual New Relic key
NEW_RELIC_LICENSE_KEY="YOUR_LICENSE_KEY"

sudo curl -o /etc/yum.repos.d/newrelic-infra.repo https://download.newrelic.com/infrastructure_agent/linux/yum/el/7/x86_64/newrelic-infra.repo
sudo yum install -y newrelic-infra
sudo bash -c "echo license_key: $NEW_RELIC_LICENSE_KEY > /etc/newrelic-infra.yml"
sudo systemctl enable newrelic-infra
sudo systemctl start newrelic-infra
```

**💡 Get license key** from: \[New Relic Account → Account Settings → API Keys]

---

## 📦 Step 6️⃣: Access & Test

| Service       | URL                            | Default Credentials |
| :------------ | :----------------------------- | :------------------ |
| Prometheus    | `http://<EC2-IP>:9090`         | n/a                 |
| Grafana       | `http://<EC2-IP>:3000`         | admin / admin       |
| Node Exporter | `http://<EC2-IP>:9100/metrics` | n/a                 |
| New Relic     | Go to your New Relic dashboard |                     |

**Grafana Configuration:**

* Add new **Data Source** → **Prometheus**

  * URL: `http://localhost:9090`
* Import dashboard → Use Grafana dashboards from [Grafana.com/dashboards](https://grafana.com/grafana/dashboards)

---

## 📦 Step 7️⃣: Test Metrics

* Visit Prometheus targets page: `http://<EC2-IP>:9090/targets`
* Confirm `node_exporter` and `prometheus` are **up**
* In Grafana, create a new dashboard → query:

  ```
  node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes
  ```

---

## 📄 📖 Documentation: `README.md`

````markdown
# EC2 Observability Stack: Prometheus, Grafana, New Relic

## 📦 Overview
Monitors EC2 metrics via Prometheus, visualizes in Grafana, and reports infrastructure telemetry to New Relic.

## 📌 Services & Ports
- Prometheus: 9090
- Grafana: 3000
- Node Exporter: 9100
- New Relic agent via API

## 📜 Setup Steps
1. Launch Amazon Linux 2 EC2 with required security group.
2. SSH into EC2.
3. Run:
   ```bash
   chmod +x install_prometheus_grafana.sh
   ./install_prometheus_grafana.sh
````

4. Setup `prometheus.yml`
5. Install Node Exporter
6. Run:

   ```bash
   chmod +x install_newrelic.sh
   ./install_newrelic.sh
   ```

## 📊 Test & Validate

* Check Prometheus: http\://<EC2-IP>:9090
* Check Grafana: http\://<EC2-IP>:3000
* New Relic Dashboard

```

---

## ✅ Final Notes

- This project can be extended with **docker-compose**, **CloudWatch agent**, or **Grafana cloud plugins**.
- You can turn this into a Terraform/CloudFormation automation too if needed.

Excellent — let’s properly extend your project with working code snippets for each of those options:

---

## 📦 1️⃣ Docker Compose Observability Stack

Let’s containerize **Prometheus**, **Grafana**, and **Node Exporter** with **docker-compose**

### 📁 `docker-compose.yml`

```yaml
version: '3'

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

  node_exporter:
    image: prom/node-exporter
    container_name: node_exporter
    ports:
      - "9100:9100"
```

### 📁 `prometheus.yml`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['prometheus:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node_exporter:9100']
```

**Run:**

```bash
docker-compose up -d
```

---

## 📦 2️⃣ Install CloudWatch Agent on EC2

### 📜 Install and Configure

```bash
sudo yum install -y amazon-cloudwatch-agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

Answer prompts for metrics and logs collection.

**Start agent**

```bash
sudo systemctl start amazon-cloudwatch-agent
sudo systemctl enable amazon-cloudwatch-agent
```

---

## 📦 3️⃣ Grafana Cloud Plugin Example

If you’re using **Grafana OSS**, you can install plugins like this:

```bash
grafana-cli plugins install grafana-clock-panel
grafana-cli plugins install grafana-piechart-panel
systemctl restart grafana-server
```

**List installed plugins**

```bash
grafana-cli plugins ls
```

---

## 📦 4️⃣ Terraform Automation for EC2 + Security Group

### 📁 `main.tf`

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "observability_ec2" {
  ami                         = "ami-0c94855ba95c71c99" # Amazon Linux 2
  instance_type               = "t2.micro"
  key_name                    = "your-key"
  associate_public_ip_address = true

  vpc_security_group_ids = [aws_security_group.monitoring_sg.id]

  tags = {
    Name = "Observability-EC2"
  }
}

resource "aws_security_group" "monitoring_sg" {
  name        = "monitoring-sg"
  description = "Allow SSH, Prometheus, Grafana, Node Exporter"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 9090
    to_port     = 9090
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 3000
    to_port     = 3000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 9100
    to_port     = 9100
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

**Commands**

```bash
terraform init
terraform apply -auto-approve
```

---

## 📦 5️⃣ CloudFormation Template

### 📁 `ec2-monitoring-stack.yaml`

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: EC2 instance for Prometheus, Grafana and Node Exporter

Resources:
  MonitoringInstance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-0c94855ba95c71c99
      KeyName: your-key
      SecurityGroupIds:
        - !Ref MonitoringSecurityGroup
      Tags:
        - Key: Name
          Value: ObservabilityEC2

  MonitoringSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow monitoring ports
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 9090
          ToPort: 9090
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 3000
          ToPort: 3000
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 9100
          ToPort: 9100
          CidrIp: 0.0.0.0/0
      SecurityGroupEgress:
        - IpProtocol: -1
          FromPort: 0
          ToPort: 0
          CidrIp: 0.0.0.0/0
```

**Deploy:**

```bash
aws cloudformation create-stack --stack-name observability-stack --template-body file://ec2-monitoring-stack.yaml
```

---

## ✅ Summary

You now have:

* 🐳 **Docker Compose** based deployment
* ☁️ **CloudWatch agent** for native EC2 monitoring
* 📊 **Grafana Cloud plugins** installer
* 📜 **Terraform IaC** for provisioning EC2 with security group
* 📜 **CloudFormation template** for infrastructure automation
