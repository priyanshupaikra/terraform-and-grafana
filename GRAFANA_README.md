# Grafana - Complete Setup & Guide

![Grafana](https://img.shields.io/badge/Grafana-v10.0+-blue) ![License](https://img.shields.io/badge/License-AGPL-green) ![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen)

## Table of Contents

- [Introduction](#introduction)
- [Installation](#installation)
- [Getting Started](#getting-started)
- [Data Sources](#data-sources)
- [Creating Dashboards](#creating-dashboards)
- [Alerts & Notifications](#alerts--notifications)
- [Panel Types](#panel-types)
- [Advanced Features](#advanced-features)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

---

## Introduction

### What is Grafana?

Grafana is an open-source platform for **real-time data visualization** and **monitoring**. It allows you to:
- Query data from multiple sources
- Visualize metrics in real-time
- Create interactive dashboards
- Set up alerts and notifications
- Monitor applications and infrastructure

### Key Features

- **Multi-Source**: Support for 50+ data sources (Prometheus, InfluxDB, Elasticsearch, etc.)
- **Rich Visualizations**: 30+ panel types for different data representations
- **Alerting**: Threshold-based alerts with multiple notification channels
- **Dashboard Management**: Create, organize, and share dashboards
- **User Management**: Role-based access control (RBAC)
- **Plugins**: Extend functionality with community plugins
- **Templating**: Create dynamic, reusable dashboards

### Use Cases

- **Infrastructure Monitoring**: CPU, memory, disk usage
- **Application Performance**: Request rates, latency, errors
- **Database Monitoring**: Queries, connections, performance
- **Business Metrics**: User activity, sales, conversions
- **IoT Monitoring**: Sensor data and device health
- **Log Analysis**: Aggregate and visualize log data

---

## Installation

### Option 1: Docker (Recommended for Quick Start)

#### Using Docker

```bash
# Pull and run Grafana image
docker run -d \
  -p 3000:3000 \
  --name grafana \
  -e GF_SECURITY_ADMIN_PASSWORD=admin \
  grafana/grafana:latest

# Access Grafana
# URL: http://localhost:3000
# Username: admin
# Password: admin
```

#### Using Docker Compose

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_INSTALL_PLUGINS=grafana-worldmap-panel,grafana-piechart-panel
      - GF_SERVER_ROOT_URL=http://localhost:3000
    volumes:
      - grafana_storage:/var/lib/grafana
      - ./provisioning/dashboards:/etc/grafana/provisioning/dashboards
      - ./provisioning/datasources:/etc/grafana/provisioning/datasources
    networks:
      - monitoring

  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_storage:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    networks:
      - monitoring

  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/'
      - '--path.sysfs=/host/sys'
    networks:
      - monitoring

volumes:
  grafana_storage:
  prometheus_storage:

networks:
  monitoring:
    driver: bridge
```

Run with:

```bash
docker-compose up -d
```

### Option 2: Linux Installation

#### Ubuntu/Debian

```bash
# Add Grafana repository
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"

# Add Grafana GPG key
sudo apt-get install -y gnupg
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

# Install Grafana
sudo apt-get update
sudo apt-get install -y grafana-server

# Start Grafana service
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

# Verify installation
sudo systemctl status grafana-server
```

#### CentOS/RHEL

```bash
# Add Grafana repository
cat > /etc/yum.repos.d/grafana.repo << EOF
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
EOF

# Install Grafana
sudo yum install grafana-server

# Start service
sudo systemctl start grafana-server
sudo systemctl enable grafana-server
```

### Verify Installation

```bash
# Check Grafana is running
curl http://localhost:3000

# Default credentials
# Username: admin
# Password: admin
```

---

## Getting Started

### Step 1: Access Grafana

1. Open browser: `http://localhost:3000`
2. Login with default credentials:
   - Username: `admin`
   - Password: `admin`
3. You'll be prompted to change the password

### Step 2: Change Admin Password

1. Click on your profile icon (top right)
2. Select "Change password"
3. Enter current and new password
4. Click "Change password"

### Step 3: Add Data Source

1. Go to **Configuration** (gear icon) → **Data Sources**
2. Click **Add data source**
3. Choose your data source type (Prometheus, InfluxDB, etc.)
4. Configure connection details
5. Click **Save & test**

### Step 4: Create Your First Dashboard

1. Click **+** icon → **Dashboard**
2. Click **Add a new panel**
3. Configure the query from your data source
4. Choose visualization type
5. Set title and description
6. Click **Save**

---

## Data Sources

### Supported Data Sources

Grafana supports 50+ data sources. Common ones:

#### 1. **Prometheus**

Prometheus is a time-series database perfect for metrics collection.

**Configuration:**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
  
  - job_name: 'docker'
    static_configs:
      - targets: ['localhost:9323']
```

**Add to Grafana:**

1. Go to Configuration → Data Sources
2. Click "Add data source"
3. Select **Prometheus**
4. Set URL: `http://prometheus:9090`
5. Click "Save & test"

#### 2. **InfluxDB**

Time-series database for metrics and events.

```bash
# Docker run
docker run -d \
  --name influxdb \
  -p 8086:8086 \
  -e INFLUXDB_DB=mydb \
  -e INFLUXDB_ADMIN_USER=admin \
  -e INFLUXDB_ADMIN_PASSWORD=password \
  influxdb:latest
```

**Add to Grafana:**

1. Data Sources → Add data source
2. Select **InfluxDB**
3. Set URL: `http://influxdb:8086`
4. Enter database name, username, password
5. Save & test

#### 3. **Elasticsearch**

Search and analytics engine.

```bash
# Docker run
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -e discovery.type=single-node \
  docker.elastic.co/elasticsearch/elasticsearch:latest
```

#### 4. **MySQL/PostgreSQL**

Relational databases.

**Add to Grafana:**

1. Data Sources → Add data source
2. Select **MySQL** or **PostgreSQL**
3. Set host, port, database, user, password
4. Save & test

#### 5. **CloudWatch**

AWS metrics and logs.

**Configuration:**

1. Data Sources → Add data source
2. Select **CloudWatch**
3. Add AWS Access Key ID and Secret Access Key
4. Select default region
5. Save & test

---

## Creating Dashboards

### Dashboard Structure

```
Dashboard
├── Row 1
│   ├── Panel 1 (Graph)
│   ├── Panel 2 (Stat)
│   └── Panel 3 (Gauge)
└── Row 2
    ├── Panel 4 (Table)
    └── Panel 5 (Heatmap)
```

### Step-by-Step Dashboard Creation

#### Step 1: Create New Dashboard

```bash
# Via UI: + icon → Dashboard → New dashboard
# Or: Dashboard → Create → Dashboard
```

#### Step 2: Add Panel

1. Click **Add panel** or **Add a new panel**
2. Configure data query:
   - Select data source
   - Enter query (depends on data source)
   - Set time range
   - Configure aggregation/grouping

#### Step 3: Choose Visualization

Select from 30+ panel types:
- **Graph/Timeseries**: Line graphs for metrics over time
- **Stat**: Single value display
- **Gauge**: Visual gauge representation
- **Table**: Tabular data
- **Bar Gauge**: Horizontal/vertical bars
- **Heatmap**: Color-coded grid
- **Pie Chart**: Proportional representation
- **Histogram**: Distribution visualization
- **Logs**: Log entries display

#### Step 4: Configure Panel Options

```
Panel Options:
├── Title: "CPU Usage"
├── Description: "CPU usage percentage"
├── Unit: "percent (0-100)"
├── Thresholds: [50 (yellow), 80 (red)]
├── Color scheme: "Threshold"
├── Min/Max values
└── Decimals: 2
```

#### Step 5: Save Dashboard

1. Click **Save dashboard**
2. Enter dashboard name
3. Choose folder
4. Add tags for organization
5. Click **Save**

### Example Dashboard: System Monitoring

```json
{
  "dashboard": {
    "title": "System Monitoring",
    "tags": ["system", "monitoring"],
    "timezone": "UTC",
    "panels": [
      {
        "id": 1,
        "title": "CPU Usage",
        "type": "timeseries",
        "targets": [
          {
            "expr": "rate(node_cpu_seconds_total{mode=\"user\"}[5m]) * 100"
          }
        ]
      },
      {
        "id": 2,
        "title": "Memory Usage",
        "type": "gauge",
        "targets": [
          {
            "expr": "(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100"
          }
        ]
      },
      {
        "id": 3,
        "title": "Disk Usage",
        "type": "timeseries",
        "targets": [
          {
            "expr": "(1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) * 100"
          }
        ]
      }
    ]
  }
}
```

### Templating (Dynamic Dashboards)

Variables make dashboards reusable.

#### Add Variable

1. Dashboard settings → Variables → Add variable
2. Configure:
   - **Name**: Variable name (e.g., `instance`)
   - **Type**: Query, Constant, Custom, etc.
   - **Data source**: Where to get values
   - **Query**: PromQL or SQL query
   - **Refresh**: On dashboard load, on time range change

#### Example: Instance Variable

```
Name: instance
Type: Query
Data source: Prometheus
Query: label_values(node_uname_info, instance)
```

#### Use in Panels

```promql
# Without variable
rate(node_cpu_seconds_total[5m])

# With variable
rate(node_cpu_seconds_total{instance="$instance"}[5m])
```

---

## Alerts & Notifications

### Creating Alerts

#### Step 1: Configure Alert Rule

1. Go to **Alerting** → **Alert rules**
2. Click **Create alert rule**
3. Configure:
   - **Rule name**: Descriptive name
   - **Data source**: Where to query
   - **Query**: Metric to monitor
   - **Condition**: When to trigger
   - **For**: How long condition must be true
   - **Annotations**: Alert details

#### Step 2: Set Conditions

```
Alert triggers when:
└── avg(node_cpu_seconds_total) > 0.8
    └── for 5 minutes
```

#### Step 3: Add Notification Channel

1. Go to **Alerting** → **Notification channels**
2. Click **New channel**
3. Choose channel type:
   - Email
   - Slack
   - PagerDuty
   - Webhook
   - Teams
   - Opsgenie
   - etc.
4. Configure settings
5. Test connection
6. Save

### Example Alert Rules

#### High CPU Alert

```yaml
name: HighCPUUsage
description: "Alert when CPU usage exceeds 80%"
condition: |
  avg(rate(node_cpu_seconds_total{mode="user"}[5m])) > 0.8
for: 5m
annotations:
  summary: "High CPU usage detected"
  description: "CPU usage is {{ $value }}%"
labels:
  severity: warning
```

#### High Memory Alert

```yaml
name: HighMemoryUsage
description: "Alert when memory usage exceeds 85%"
condition: |
  (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) > 0.85
for: 10m
annotations:
  summary: "High memory usage detected"
  description: "Memory usage is {{ $value }}%"
labels:
  severity: critical
```

#### Disk Full Alert

```yaml
name: DiskAlmostFull
description: "Alert when disk is 90% full"
condition: |
  (1 - (node_filesystem_avail_bytes / node_filesystem_size_bytes)) > 0.9
for: 5m
annotations:
  summary: "Disk almost full"
  description: "Disk {{ $labels.device }} is {{ $value }}% full"
labels:
  severity: critical
```

### Notification Channels

#### Email

```
Type: Email
Recipients: ops@example.com
From: grafana@example.com
SMTP Server: smtp.example.com
SMTP Port: 587
```

#### Slack

```
Type: Slack
Webhook URL: https://hooks.slack.com/services/YOUR/WEBHOOK/URL
Channel: #alerts
Username: Grafana
```

#### Webhook

```
Type: Webhook
URL: http://example.com/webhook
HTTP Method: POST
Headers:
  Authorization: Bearer token
```

---

## Panel Types

### 1. Graph / Timeseries

Best for: Metrics over time

```
Configuration:
├── Query: Time-series metric
├── Axis:
│   ├── Left Y-axis: Value range
│   └── Right Y-axis: Optional second metric
├── Legend: Show/hide, position
├── Stacking: None, Normal, Percent
└── Fill: Opacity level
```

### 2. Stat

Best for: Single important value

```
Configuration:
├── Query: Metric
├── Value options:
│   ├── Calculation: Last, Mean, Max, Min
│   └── Decimals: Number precision
├── Thresholds: Warning/Critical values
└── Color scheme: Value or Threshold based
```

### 3. Gauge

Best for: Ranges (0-100%)

```
Configuration:
├── Query: Metric
├── Gauge options:
│   ├── Min value: 0
│   ├── Max value: 100
│   └── Gauge type: Gauge, Sphere
├── Thresholds: [50, 80]
└── Unit: percent, short, etc.
```

### 4. Table

Best for: Structured data

```
Configuration:
├── Query: Multi-valued metric
├── Columns:
│   ├── Name: Column header
│   ├── Value: Field name
│   └── Format: String, Number, Date
├── Sort: By column, ascending/descending
└── Pagination: Rows per page
```

### 5. Bar Gauge

Best for: Comparison

```
Configuration:
├── Query: Multiple values
├── Orientation: Horizontal/Vertical
├── Display style: LCD, Gradient
├── Thresholds: Color ranges
└── Min/Max: Value ranges
```

### 6. Heatmap

Best for: Distribution over time

```
Configuration:
├── Query: Time-series with histogram bucket
├── Bucket size: Y-axis buckets
├── Color scheme: Gradual, spectral
└── Legend: Show/hide
```

### 7. Pie Chart

Best for: Proportions

```
Configuration:
├── Query: Multi-valued metric
├── Display style: Pie, Donut
├── Legend: Position, values
├── Tooltips: Percentage, value
└── Unit: auto, percent, short
```

---

## Advanced Features

### 1. Dashboard Variables

#### Query Variable

```
Name: region
Type: Query
Data source: Prometheus
Query: label_values(metric, region)
Multi-value: Yes
Include all: Yes
```

#### Constant Variable

```
Name: environment
Type: Constant
Value: production
```

#### Custom Variable

```
Name: color
Type: Custom
Values: red, green, blue
```

### 2. Provisioning

Automate dashboard and datasource creation.

**Datasource Provisioning** (`provisioning/datasources/prometheus.yml`):

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true

  - name: InfluxDB
    type: influxdb
    access: proxy
    url: http://influxdb:8086
    database: mydb
    user: admin
    secureJsonData:
      password: password
```

**Dashboard Provisioning** (`provisioning/dashboards/dashboard-provider.yml`):

```yaml
apiVersion: 1

providers:
  - name: 'My Dashboards'
    orgId: 1
    folder: 'Dashboards'
    type: file
    disableDeletion: false
    updateIntervalSeconds: 10
    allowUiUpdates: true
    options:
      path: /etc/grafana/provisioning/dashboards
```

### 3. Organization & Teams

**Create Organization**

1. Administration → Organizations
2. Click "New organization"
3. Enter name
4. Click "Create"

**Create Team**

1. Administration → Teams
2. Click "New team"
3. Enter team name
4. Add members
5. Set permissions

### 4. User Management

**Add User**

1. Administration → Users
2. Click "Invite"
3. Enter email
4. Choose role (Admin, Editor, Viewer)
5. Send invite

**Roles:**
- **Admin**: Full access
- **Editor**: Create/edit dashboards
- **Viewer**: Read-only access

### 5. Plugins

**Install Plugin**

```bash
# Via CLI
grafana-cli plugins install grafana-piechart-panel

# Via Docker
docker run -e GF_INSTALL_PLUGINS=grafana-piechart-panel grafana/grafana
```

**Popular Plugins:**
- grafana-worldmap-panel (Maps)
- grafana-piechart-panel (Pie charts)
- grafana-clock-panel (Clock)
- grafana-singlestat-panel (Single stats)
- grafana-influxdb-flux-datasource (InfluxDB Flux)

---

## Best Practices

### Dashboard Design

✅ **DO:**
- Keep dashboards focused on specific aspects
- Use descriptive titles and descriptions
- Organize panels logically (top-to-bottom, left-to-right)
- Use consistent colors and formatting
- Add annotations for events
- Document dashboard purpose
- Share dashboards appropriately

❌ **DON'T:**
- Overcrowd dashboards with too many panels
- Use irrelevant visualizations
- Mix multiple unrelated metrics
- Create unnecessarily complex queries
- Forget to set time ranges
- Ignore panel titles

### Query Performance

✅ **DO:**
- Use appropriate time ranges
- Aggregate metrics at scrape time
- Use rate() for counters
- Set recording rules for complex queries
- Monitor query performance
- Use caching where applicable

❌ **DON'T:**
- Query very long time ranges unnecessarily
- Use aggregations that are too granular
- Create expensive join operations
- Ignore datasource limits
- Query raw data when aggregates exist

### Alerting Best Practices

✅ **DO:**
- Set meaningful alert thresholds
- Include context in alert messages
- Test alerts regularly
- Document alert reasoning
- Use appropriate channels
- Set escalation policies
- Monitor alert noise

❌ **DON'T:**
- Set trivial thresholds that always trigger
- Create alerts without clear action items
- Ignore alert fatigue
- Send duplicate notifications
- Create meaningless alert messages
- Set unrealistic thresholds

### Security

✅ **DO:**
- Change default admin password
- Use strong passwords
- Enable SSL/TLS
- Restrict network access
- Use environment variables for secrets
- Enable authentication
- Audit user access
- Regular backups

❌ **DON'T:**
- Leave default credentials
- Store passwords in config files
- Use HTTP in production
- Allow public access
- Share admin credentials
- Ignore security updates

---

## Troubleshooting

### Common Issues

#### Issue 1: Grafana Won't Start

```bash
# Check logs
docker logs grafana

# Check port availability
lsof -i :3000

# Check permissions
sudo chown -R grafana:grafana /var/lib/grafana

# Restart service
sudo systemctl restart grafana-server
```

#### Issue 2: No Data Displayed

```bash
# Test data source connection
# In Grafana: Data Sources → [Your Source] → Save & Test

# Check data source query
# In Prometheus:
# curl http://localhost:9090/api/v1/query?query=your_metric

# Verify time range has data
# Dashboard → Time range adjustment

# Check metric exists
# Prometheus UI: http://localhost:9090/graph
```

#### Issue 3: Alerts Not Firing

```bash
# Check alert rule
# Alerting → Alert rules → [Your Alert] → Edit

# Verify notification channel
# Alerting → Notification channels → [Your Channel] → Test

# Check logs
docker logs grafana | grep alert

# Verify data exists
# Query in panel to confirm metric exists
```

#### Issue 4: Performance Issues

```
Solutions:
├── Increase resource allocation
├── Optimize queries (use rate(), aggregation)
├── Set appropriate scrape intervals
├── Use datasource caching
├── Reduce dashboard panel count
├── Archive old dashboards
└── Monitor Grafana metrics
```

#### Issue 5: Disk Space Issues

```bash
# Check disk usage
docker exec grafana du -sh /var/lib/grafana

# Clean old data
docker exec grafana grafana-cli admin reset-admin-password newpassword

# Increase volume size
# Docker: Update volume mount size
```

---

## Essential Commands

### Docker Commands

```bash
# Pull image
docker pull grafana/grafana:latest

# Run container
docker run -d \
  -p 3000:3000 \
  --name grafana \
  grafana/grafana:latest

# View logs
docker logs -f grafana

# Stop container
docker stop grafana

# Remove container
docker rm grafana

# Execute command
docker exec grafana grafana-cli plugins list

# Check resource usage
docker stats grafana
```

### Linux Commands

```bash
# Start service
sudo systemctl start grafana-server

# Stop service
sudo systemctl stop grafana-server

# Restart service
sudo systemctl restart grafana-server

# Check status
sudo systemctl status grafana-server

# View logs
sudo tail -f /var/log/grafana/grafana.log

# Enable on startup
sudo systemctl enable grafana-server
```

### CLI Commands

```bash
# Reset admin password
grafana-cli admin reset-admin-password newpassword

# List plugins
grafana-cli plugins list

# Install plugin
grafana-cli plugins install grafana-piechart-panel

# Update plugin
grafana-cli plugins update grafana-piechart-panel

# Uninstall plugin
grafana-cli plugins remove grafana-piechart-panel
```

---

## Configuration

### Environment Variables

```bash
# Server
GF_SERVER_HTTP_PORT=3000
GF_SERVER_PROTOCOL=http
GF_SERVER_ROOT_URL=http://localhost:3000

# Security
GF_SECURITY_ADMIN_USER=admin
GF_SECURITY_ADMIN_PASSWORD=admin
GF_SECURITY_SECRET_KEY=secretkey

# Database
GF_DATABASE_TYPE=sqlite3
GF_DATABASE_HOST=localhost
GF_DATABASE_NAME=grafana
GF_DATABASE_USER=grafana
GF_DATABASE_PASSWORD=password

# Auth
GF_AUTH_ANONYMOUS_ENABLED=false
GF_AUTH_GITHUB_ENABLED=false

# SMTP
GF_SMTP_ENABLED=true
GF_SMTP_HOST=smtp.example.com:587
GF_SMTP_USER=user@example.com
GF_SMTP_PASSWORD=password
```

### Configuration File

Location: `/etc/grafana/grafana.ini`

```ini
[server]
http_port = 3000
protocol = http
root_url = http://localhost:3000

[security]
admin_user = admin
admin_password = admin
secret_key = secretkey

[database]
type = sqlite3
path = /var/lib/grafana/grafana.db

[auth]
disable_login_form = false

[smtp]
enabled = true
host = smtp.example.com:587
user = user@example.com
password = password
from_name = Grafana
from_address = grafana@example.com
```

---

## Resources

### Official Documentation
- [Grafana Official Docs](https://grafana.com/docs/)
- [Grafana GitHub](https://github.com/grafana/grafana)
- [Grafana Community](https://community.grafana.com/)

### Data Sources
- [Prometheus](https://prometheus.io/)
- [InfluxDB](https://www.influxdata.com/)
- [Elasticsearch](https://www.elastic.co/)
- [PostgreSQL](https://www.postgresql.org/)
- [MySQL](https://www.mysql.com/)

### Learning Resources
- [Grafana Tutorials](https://grafana.com/tutorials/)
- [Grafana Academy](https://grafana.com/grafana/academy/)
- [YouTube Channel](https://www.youtube.com/c/Grafana)

### Community Plugins
- [Grafana Plugin Registry](https://grafana.com/grafana/plugins/)
- [Official Plugins](https://grafana.com/grafana/plugins/?type=app)

---

## Monitoring Stack Diagram

```
Data Sources
├── Prometheus (metrics)
├── InfluxDB (time-series)
├── Elasticsearch (logs)
└── MySQL (relational)
         ↓
    Grafana (visualization)
         ↓
    Dashboards & Alerts
         ↓
    Notifications
├── Email
├── Slack
├── PagerDuty
└── Webhook
```

---

## Quick Start Summary

```bash
# 1. Run with Docker Compose
docker-compose up -d

# 2. Access Grafana
# Open: http://localhost:3000
# Login: admin / admin

# 3. Change password
# Profile → Change password

# 4. Add Prometheus datasource
# Configuration → Data Sources → Add → Prometheus
# URL: http://prometheus:9090
# Save & Test

# 5. Create dashboard
# + → Dashboard → Add Panel
# Select Prometheus as data source
# Enter query: up
# Save

# 6. Set up alerts
# Alerting → New Alert
# Set condition and notification channel
# Save
```

---

## Next Steps

1. **Explore Dashboard Gallery**: Look at existing dashboards for inspiration
2. **Create Custom Dashboards**: Start monitoring your applications
3. **Set Up Alerts**: Configure notifications for critical metrics
4. **Learn PromQL**: Master Prometheus query language
5. **Share Dashboards**: Distribute to team members
6. **Automate Provisioning**: Use provisioning files for GitOps

---

## Support

- **Grafana Docs**: https://grafana.com/docs/
- **Community Forum**: https://community.grafana.com/
- **GitHub Issues**: https://github.com/grafana/grafana/issues
- **Slack Community**: https://slack.grafana.com/

---

**Happy Monitoring! 📊**

---

*Last Updated: March 15, 2026*  
*Version: 1.0.0*  
*Status: Production Ready*
