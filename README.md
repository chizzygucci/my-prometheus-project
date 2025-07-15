# Prometheus Setup with Node Exporter — Issues and Fixes

Project Summary
This setup implements a production-style Prometheus monitoring system using:

Prometheus – Metrics collection

Node Exporter – Host-level system metrics

Systemd Services – For persistent background execution

Filesystem Hierarchy Standard (FHS) – Configuration in /etc/



Directory Structure

/etc/prometheus/
├── prometheus.yml          # Main config
├── rules/
└── file_sd/                # Target discovery

/etc/systemd/system/
└── prometheus.service      # Prometheus as systemd service

/var/lib/prometheus/        # TSDB storage



 Download Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.52.0/prometheus-2.52.0.linux-amd64.tar.gz
tar -xvzf prometheus-2.52.0.linux-amd64.tar.gz
sudo mv prometheus-2.52.0.linux-amd64 /opt/prometheus

# Move binaries to a global path
sudo cp /opt/prometheus/prometheus /usr/local/bin/
sudo cp /opt/prometheus/promtool /usr/local/bin/

# Create directories
sudo mkdir -p /etc/prometheus /var/lib/prometheus

# Create systemd service file
sudo nano /etc/systemd/system/prometheus.service


# Problem 1: prometheus.service failed with exit-code 203/EXEC
 Cause:
Incorrect binary path in the systemd unit file. Prometheus was installed from a tarball and manually moved,
not installed via APT, so systemd couldn't locate it at the default /usr/bin/prometheus.


# Fix:
Update /etc/systemd/system/prometheus.service:

[Service]
User=prometheus
ExecStart=/usr/local/bin/prometheus \


# Problem 2: Node Exporter metrics not visible in Prometheus
🔍 Cause:
Node Exporter was running, but Prometheus wasn't scraping it because it wasn't added in prometheus.yml.

# Fix:
Update /etc/prometheus/prometheus.yml:

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
   
  # Problem 4: Wrong permissions on  /usr/local/bin/prometheus

   Cause:
Files copied as root without adjusting ownership. Prometheus couldn’t read/write config or TSDB directory.

# Fix:

sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
  
