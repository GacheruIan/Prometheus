# Prometheus Installation on Linux

This guide documents the step-by-step installation and configuration of Prometheus using a binary installation approach and running it as a systemd service.

Prometheus is an open-source monitoring and alerting toolkit used to collect and store time-series metrics in DevOps and cloud environments.

---

## 1. Update System Packages

```bash
sudo apt-get update
```

Purpose

Refreshes package lists

Prevents dependency conflicts during installation

---

## 2. Download Prometheus

```bash
sudo wget https://github.com/prometheus/prometheus/releases/download/v3.10.0-rc.1/prometheus-3.10.0-rc.1.linux-amd64.tar.gz
```

Purpose

Downloads the Prometheus precompiled binary

## 3. Create Prometheus Service User (Security Best Practice)

```bash
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```

Why

Prevents Prometheus from running as root

Improves system security

## 4. Create Prometheus Directories

```bash
sudo mkdir /var/lib/prometheus
sudo mkdir -p /etc/prometheus/rules
sudo mkdir -p /etc/prometheus/rules.s
sudo mkdir -p /etc/prometheus/files_sd
```

| Directory             | Purpose                      |
| --------------------- | ---------------------------- |
| `/var/lib/prometheus` | Metrics storage (TSDB)       |
| `/etc/prometheus`     | Configuration files          |
| `rules/`              | Alerting & recording rules   |
| `files_sd/`           | File-based service discovery |


## 5. Extract Archive

```bash
sudo tar xzf prometheus-3.10.0-rc.1.linux-amd64.tar.gz
```
## 6. Move Binaries to System Path

```bash
sudo mv prometheus promtool /usr/local/bin
prometheus --version
```

Why

Allows Prometheus to run globally

promtool is used for config validation



## 7. Move Configuration File

```bash
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```

## 8. Create Systemd Service

```bash
sudo tee /etc/systemd/system/prometheus.service<<EOF
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP \$MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

Purpose

Runs Prometheus as a managed Linux service

Enables restart on failure

Starts automatically on boot


## 9. Set Permissions

```bash
sudo chown -R prometheus:prometheus /etc/prometheus
sudo chmod -R 755 /etc/prometheus
sudo chown -R prometheus:prometheus /var/lib/prometheus/
```

Why

Prevents permission errors

Allows Prometheus to write metrics safely

## 10. Start and Enable Service

```bash
sudo systemctl daemon-reload
sudo systemctl start prometheus
sudo systemctl enable prometheus
```

## 11. Verify Installation

```bash
sudo systemctl status prometheus
```

## 12. Open browser:

```bash
http://SERVER_IP:9090
```
