# Prometheus Pushgateway Installation and Configuration

This guide provides step-by-step instructions to install, configure, and use Prometheus Pushgateway for pushing custom metrics.

---

## Installation

### Step 1: Download and Extract Pushgateway
Get the latest version of Pushgateway from [prometheus.io](https://prometheus.io):
```bash
wget https://github.com/prometheus/pushgateway/releases/download/v0.8.0/pushgateway-0.8.0.linux-amd64.tar.gz
tar -xvf pushgateway-0.8.0.linux-amd64.tar.gz
```

### Step 2: Create the Pushgateway User
Create a system user for Pushgateway:
```bash
useradd --no-create-home --shell /bin/false pushgateway
```

### Step 3: Move the Binary and Set Permissions
Copy the Pushgateway binary and assign the correct permissions:
```bash
cp pushgateway-0.8.0.linux-amd64/pushgateway /usr/local/bin/pushgateway
chown pushgateway:pushgateway /usr/local/bin/pushgateway
```

### Step 4: Create a Systemd Unit File
Create the `pushgateway.service` file:
```bash
cat > /etc/systemd/system/pushgateway.service << EOF
[Unit]
Description=Prometheus Pushgateway
Wants=network-online.target
After=network-online.target

[Service]
User=pushgateway
Group=pushgateway
Type=simple
ExecStart=/usr/local/bin/pushgateway

[Install]
WantedBy=multi-user.target
EOF
```

### Step 5: Reload and Start Pushgateway
Reload systemd and start the Pushgateway service:
```bash
systemctl daemon-reload
systemctl restart pushgateway
```

### Step 6: Verify Pushgateway Status
Ensure that Pushgateway is running:
```bash
systemctl status pushgateway
```

---

## Configure Prometheus

Edit Prometheus's configuration file to include the Pushgateway job:

### `/etc/prometheus/prometheus.yml`
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'pushgateway'
    honor_labels: true
    static_configs:
      - targets: ['localhost:9091']
```

Restart Prometheus to apply the changes:
```bash
systemctl restart prometheus
```

---

## Push Metrics to Pushgateway

### Bash Example
Push custom metrics to Pushgateway using a bash command:
```bash
echo "cpu_utilization 20.25" | curl --data-binary @- http://localhost:9091/metrics/job/my_custom_metrics/instance/10.20.0.1:9000/provider/hetzner
```

### Verify Metrics
Retrieve metrics from Pushgateway:
```bash
curl -L http://localhost:9091/metrics/
```
Output example:
```
# TYPE cpu_utilization untyped
cpu_utilization{instance="10.20.0.1:9000",job="my_custom_metrics",provider="hetzner"} 20.25
```

### Python Example
Push custom metrics to Pushgateway using Python:
```python
import requests

job_name = 'my_custom_metrics'
instance_name = '10.20.0.1:9000'
provider = 'hetzner'
payload_key = 'cpu_utilization'
payload_value = '21.90'

response = requests.post(
    'http://localhost:9091/metrics/job/{j}/instance/{i}/provider/{p}'.format(j=job_name, i=instance_name, p=provider),
    data='{k} {v}\n'.format(k=payload_key, v=payload_value)
)
print(response.status_code)
```

---

## Summary
With these steps, you can install Prometheus Pushgateway, configure it with Prometheus, and push custom metrics using various methods like Bash or Python. This allows Prometheus to consume the metrics for monitoring and analysis.

For more details, refer to the official [Prometheus Pushgateway documentation](https://prometheus.io/docs/).
