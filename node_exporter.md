# Node Exporter Installation (Linux)

This guide explains how to install and configure Prometheus Node Exporter as a systemd service.

Node Exporter exposes Linux system metrics such as CPU, memory, disk, and network statistics so that Prometheus can scrape and store them.

Default metrics endpoint:
http://SERVER_IP:9100/metrics

---

## 1. Download Node Exporter

```bash
sudo wget <NODE_EXPORTER_DOWNLOAD_LINK>
```
Purpose

Downloads the Node Exporter binary from GitHub releases.

## 2. Extract Archive

```bash
sudo tar xzf node_exporter-*.tar.gz
```

Purpose

Extracts the Node Exporter binary from the compressed archive.

## 3. Test Node Exporter Manually

```bash
./node_exporter
```

Result

Starts Node Exporter

Exposes metrics on port 9100

Verify:

```bash
http://SERVER_IP:9100/metrics
```

This step confirms the binary works before creating a service.

## 4. Create Prometheus Service User (Security Best Practice)

```bash
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```

Why

Prevents running exporter as root

Improves production security

## 5. Create Installation Directory

```bash
sudo mkdir /var/lib/node
```


Purpose

Stores Node Exporter binary in a dedicated location.

## 6. Move Binary

```bash
sudo mv node_exporter /var/lib/node/
```
##7. Create Systemd Service

```bash
sudo nano /etc/systemd/system/node.service
```

```bash
[Unit]
Description=Prometheus Node Exporter
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/var/lib/node/node_exporter

SyslogIdentifier=prometheus_node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```

Purpose

Runs Node Exporter as a background service

Enables restart on failure

Allows automatic startup during boot


## 8. Set Permissions

```bash
sudo chown -R prometheus:prometheus /var/lib/node
sudo chmod -R 775 /var/lib/node/
```

Why

Prevents permission-related service failures

Allows Prometheus user to execute exporter

## 9. Start and Enable Service

```bash
sudo systemctl daemon-reload
sudo systemctl start node
sudo systemctl enable node
```

## 10. Verify Service

```bash
sudo systemctl status node
```

Then open:

```bash
http://SERVER_IP:9100/metrics
```

What Node Exporter Provides

CPU usage

Memory usage

Disk utilization

Network statistics

Filesystem metrics

Load average

## Connect Node Exporter to Prometheus

Add this to prometheus.yml:

```bash
scrape_configs:
  - job_name: "node_exporter"
    static_configs:
      - targets: ["SERVER_IP:9100"]
```

Then restart Prometheus:

```bash
sudo systemctl restart prometheus
```


⚠️ Common Mistakes

Forgetting to open firewall port 9100

Wrong binary path in systemd service

Permission denied errors

Not reloading systemd daemon
