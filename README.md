# Prometheus Cheat Sheet

## Installation
```bash
# Download and install
wget https://github.com/prometheus/prometheus/releases/download/v2.40.0/prometheus-2.40.0.linux-amd64.tar.gz
tar xvfz prometheus-*.tar.gz
cd prometheus-*
./prometheus --config.file=prometheus.yml

# Docker
docker run -p 9090:9090 prom/prometheus

# With custom config
docker run -p 9090:9090 -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
```

## Configuration (prometheus.yml)
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert_rules.yml"

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'web-servers'
    static_configs:
      - targets: 
        - 'web1:9100'
        - 'web2:9100'
    metrics_path: '/metrics'
    scrape_interval: 30s

  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

## PromQL Queries
```promql
# Basic queries
up
http_requests_total
http_requests_total{job="web-server"}
http_requests_total{job="web-server", method="GET"}

# Rate and increase
rate(http_requests_total[5m])
increase(http_requests_total[1h])
irate(cpu_usage_seconds_total[5m])

# Aggregation
sum(http_requests_total)
sum by (job) (http_requests_total)
avg(cpu_usage_percent) by (instance)
max(memory_usage_bytes) by (job)
count(up == 1)

# Mathematical operations
cpu_usage_percent > 80
memory_usage_bytes / 1024 / 1024  # Convert to MB
(cpu_user + cpu_system) / cpu_total * 100

# Time functions
hour()
day_of_week()
time()
timestamp(up)

# Prediction
predict_linear(cpu_usage_percent[1h], 3600)  # Predict 1 hour ahead
```

## Alert Rules (alert_rules.yml)
```yaml
groups:
  - name: system_alerts
    rules:
      - alert: HighCPUUsage
        expr: cpu_usage_percent > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}% on {{ $labels.instance }}"

      - alert: MemoryUsageHigh
        expr: memory_usage_percent > 90
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Memory usage high on {{ $labels.instance }}"

      - alert: DiskSpaceLow
        expr: disk_free_percent < 10
        for: 5m
        labels:
          severity: critical
```

## Common Exporters
```bash
# Node Exporter (system metrics)
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar xvfz node_exporter-*.tar.gz
./node_exporter

# MySQL Exporter
docker run -d -p 9104:9104 -e DATA_SOURCE_NAME="user:password@(hostname:3306)/" prom/mysqld-exporter

# Redis Exporter
docker run -d -p 9121:9121 oliver006/redis_exporter --redis.addr=redis://redis-host:6379

# Blackbox Exporter (HTTP/TCP probing)
docker run -p 9115:9115 prom/blackbox-exporter
```

## Service Discovery
```yaml
# File-based service discovery
- job_name: 'file-sd'
  file_sd_configs:
    - files:
      - '/etc/prometheus/targets/*.json'
      refresh_interval: 10s

# DNS service discovery
- job_name: 'dns-sd'
  dns_sd_configs:
    - names: ['_prometheus._tcp.example.com']

# EC2 service discovery
- job_name: 'ec2-sd'
  ec2_sd_configs:
    - region: us-west-2
      port: 9100
      filters:
        - name: tag:monitoring
          values: ['true']
```

## Recording Rules
```yaml
groups:
  - name: recording_rules
    rules:
      - record: node:cpu_utilisation:rate5m
        expr: 1 - avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) by (instance)
      
      - record: node:memory_utilisation
        expr: 1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)
```

## API Queries
```bash
# Query API
curl 'http://localhost:9090/api/v1/query?query=up'
curl 'http://localhost:9090/api/v1/query_range?query=up&start=2023-01-01T00:00:00Z&end=2023-01-01T23:59:59Z&step=1h'

# Metadata
curl 'http://localhost:9090/api/v1/label/__name__/values'
curl 'http://localhost:9090/api/v1/series?match[]=up'
```

## Official Links
- [Prometheus Documentation](https://prometheus.io/docs/)
- [PromQL Documentation](https://prometheus.io/docs/prometheus/latest/querying/)
- [Exporters and Integrations](https://prometheus.io/docs/instrumenting/exporters/)