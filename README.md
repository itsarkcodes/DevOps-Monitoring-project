<div align="center">
  <br />
    <a href="https://youtu.be/FTH6Dn3AyIQ" target="_blank">
      <img src="https://github.com/user-attachments/assets/87c6b674-ea88-48b1-a7d5-d598801dfd51" alt="Project Banner">
    </a>
  <br />

  <h2 align="center">DevOps Monitoring Project</h3>

   <div align="center">
     Complete project to build a monitoring solution with Prometheus, Node exporter, Blackbox Exporter, Alert Manager to Monitor a website
    </div>
</div>

## 📋 <a name="table">Table of Contents</a>

1. 🤖 [Introduction](#introduction)
2. ⚙️ [Stack](#tech-stack)
3. 🔋 [Features](#features)
4. 🤸 [Steps](#steps)
5. 🚀 [More](#more)

## <a name="introduction">🤖 Introduction</a>

The DevOps Monitoring Project utilizes Prometheus, Alert Manager, Blackbox Exporter, and Node Exporter to monitor various system and service metrics. This project is designed to alert users via email notifications when specific conditions are met, ensuring prompt response to potential issues.

## <a name="tech-stack">⚙️ Stack</a>
- Prometheus: An open-source monitoring and alerting toolkit.
- Alert Manager: Manages alerts sent by Prometheus.
- Blackbox Exporter: Allows Prometheus to probe endpoints over HTTP, HTTPS, DNS, TCP, ICMP, and gRPC.
- Node Exporter: Collects hardware and OS metrics

## <a name="features">🔋 Features </a>
- Instance Monitoring: Alerts when a VM instance is down.
- Website Monitoring: Alerts when a monitored website is unreachable.
- Service Monitoring: Alerts when a specific service (e.g., Node Exporter) is unavailable.
- Resource Monitoring: Alerts for high CPU load, out of memory, and no disk space conditions.
- Email Notifications: Sends detailed alert notifications via email.

## <a name="steps">🤸 Steps</a>
### We need 2 VM. 
1. One for Monitoring Tool hosting
2. Actual website hosting

#### VM-1 (Node Exporter)
1. **Download Node Exporter**
   ```bash
   wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
   ```

2. **Extract Node Exporter**
   ```bash
   tar xvfz node_exporter-1.8.1.linux-amd64.tar.gz
   ```

3. **Start Node Exporter**
   ```bash
   cd node_exporter-1.8.1.linux-amd64
   ./node_exporter &
   ```

#### VM-2 (Prometheus, Alertmanager, Blackbox Exporter)

##### Prometheus
1. **Download Prometheus**
   ```bash
   wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
   ```

2. **Extract Prometheus**
   ```bash
   tar xvfz prometheus-2.52.0.linux-amd64.tar.gz
   ```

3. **Start Prometheus**
   ```bash
   cd prometheus-2.52.0.linux-amd64
   ./prometheus --config.file=prometheus.yml &
   ```

##### Alertmanager
1. **Download Alertmanager**
   ```bash
   wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
   ```

2. **Extract Alertmanager**
   ```bash
   tar xvfz alertmanager-0.27.0.linux-amd64.tar.gz
   ```

3. **Start Alertmanager**
   ```bash
   cd alertmanager-0.27.0.linux-amd64
   ./alertmanager --config.file=alertmanager.yml &
   ```

##### Blackbox Exporter
1. **Download Blackbox Exporter**
   ```bash
   wget https://github.com/prometheus/blackbox_exporter/releases/download/v0.25.0/blackbox_exporter-0.25.0.linux-amd64.tar.gz
   ```

2. **Extract Blackbox Exporter**
   ```bash
   tar xvfz blackbox_exporter-0.25.0.linux-amd64.tar.gz
   ```

3. **Start Blackbox Exporter**
   ```bash
   cd blackbox_exporter-0.25.0.linux-amd64
   ./blackbox_exporter &
   ```

### Notes:
- The `&` at the end of each command ensures the process runs in the background.
- Ensure that you have configured the `prometheus.yml` and `alertmanager.yml` configuration files correctly before starting the services.
- Adjust the firewall and security settings to allow the necessary ports (typically 9090 for Prometheus, 9093 for Alertmanager, 9115 for Blackbox Exporter, and 9100 for Node Exporter) to be accessible.

---

# Prometheus and Alertmanager Configuration

## Prometheus Configuration (`prometheus.yml`)

### Global Configuration
```yaml
global:
  scrape_interval: 15s                # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s            # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).
```

### Alertmanager Configuration
```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 'localhost:9093'          # Alertmanager endpoint
```

### Rule Files
```yaml
rule_files:
   - "alert_rules.yml"                # Path to alert rules file
  # - "second_rules.yml"              # Additional rule files can be added here
```

### Scrape Configuration
#### Prometheus Itself
```yaml
scrape_configs:
  - job_name: "prometheus"            # Job name for Prometheus

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]   # Target to scrape (Prometheus itself)
```

#### Node Exporter
```yaml
  - job_name: "node_exporter"         # Job name for node exporter

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["102.232.232.22:9100"]  # Target node exporter endpoint
```

#### Blackbox Exporter
```yaml
  - job_name: 'blackbox'              # Job name for blackbox exporter
    metrics_path: /probe              # Path for blackbox probe
    params:
      module: [http_2xx]              # Module to look for HTTP 200 response
    static_configs:
      - targets:
        - http://prometheus.io        # HTTP target
        - https://prometheus.io       # HTTPS target
        - http://3.110.195.114:8080/  # HTTP target with port 8080
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 13.235.248.225:9115  # Blackbox exporter address
```

## Alert Rules Configuration (`alert_rules.yml`)

### Alert Rules Group
```yaml
groups:
- name: alert_rules                   # Name of the alert rules group
  rules:
    - alert: InstanceDown
      expr: up == 0                   # Expression to detect instance down
      for: 1m
      labels:
        severity: "critical"
      annotations:
        summary: "Endpoint {{ $labels.instance }} down"
        description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minute."

    - alert: WebsiteDown
      expr: probe_success == 0        # Expression to detect website down
      for: 1m
      labels:
        severity: critical
      annotations:
        description: The website at {{ $labels.instance }} is down.
        summary: Website down

    - alert: HostOutOfMemory
      expr: node_memory_MemAvailable / node_memory_MemTotal * 100 < 25  # Expression to detect low memory
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Host out of memory (instance {{ $labels.instance }})"
        description: "Node memory is filling up (< 25% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: HostOutOfDiskSpace
      expr: (node_filesystem_avail{mountpoint="/"} * 100) / node_filesystem_size{mountpoint="/"} < 50  # Expression to detect low disk space
      for: 1s
      labels:
        severity: warning
      annotations:
        summary: "Host out of disk space (instance {{ $labels.instance }})"
        description: "Disk is almost full (< 50% left)\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: HostHighCpuLoad
      expr: (sum by (instance) (irate(node_cpu{job="node_exporter_metrics",mode="idle"}[5m]))) > 80  # Expression to detect high CPU load
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Host high CPU load (instance {{ $labels.instance }})"
        description: "CPU load is > 80%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: ServiceUnavailable
      expr: up{job="node_exporter"} == 0  # Expression to detect service unavailability
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "Service Unavailable (instance {{ $labels.instance }})"
        description: "The service {{ $labels.job }} is not available\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: HighMemoryUsage
      expr: (node_memory_Active / node_memory_MemTotal) * 100 > 90  # Expression to detect high memory usage
      for: 10m
      labels:
        severity: critical
      annotations:
        summary: "High Memory Usage (instance {{ $labels.instance }})"
        description: "Memory usage is > 90%\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"

    - alert: FileSystemFull
      expr: (node_filesystem_avail / node_filesystem_size) * 100 < 10  # Expression to detect file system almost full
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "File System Almost Full (instance {{ $labels.instance }})"
        description: "File system has < 10% free space\n  VALUE = {{ $value }}\n  LABELS: {{ $labels }}"
```

## Alertmanager Configuration (`alertmanager.yml`)

### Routing Configuration
```yaml
route:
  group_by: ['alertname']             # Group by alert name
  group_wait: 30s                     # Wait time before sending the first notification
  group_interval: 5m                  # Interval between notifications
  repeat_interval: 1h                 # Interval to resend notifications
  receiver: 'email-notifications'     # Default receiver

receivers:
- name: 'email-notifications'         # Receiver name
  email_configs:
  - to: senderEmail@gmail.com       # Email recipient
    from: mymonitoring@gmail.com              # Email sender
    smarthost: smtp.gmail.com:587     # SMTP server
    auth_username: your_email         # SMTP auth username
    auth_identity: your_email         # SMTP auth identity
    auth_password: "tsffd wqsg vvkk zoqx"  # SMTP auth password
    send_resolved: true               # Send notifications for resolved alerts
```

### Inhibition Rules
```yaml
inhibit_rules:
  - source_match:
      severity: 'critical'            # Source alert severity
    target_match:
      severity: 'warning'             # Target alert severity
    equal: ['alertname', 'dev', 'instance']  # Fields to match
```

---
## <a name="more">🚀 More</a>
For more information on the setup and configuration of Prometheus, Alert Manager, Blackbox Exporter, and Node Exporter, please refer to their official documentation:
Prometheus Documentation [here](https://prometheus.io/docs/introduction/overview/)
Alert Manager Documentation [here](https://drive.google.com/file/d/1ZmtiMilUYTp1wkiXWMFX6AUk-msE981-/view?usp=sharing)
Blackbox Exporter Documentation [here](https://drive.google.com/file/d/1ZmtiMilUYTp1wkiXWMFX6AUk-msE981-/view?usp=sharing)
Node Exporter Documentation [here](https://drive.google.com/file/d/1ZmtiMilUYTp1wkiXWMFX6AUk-msE981-/view?usp=sharing)

