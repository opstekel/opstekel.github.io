---
title: "Install Prometheus dan Grafana"
description: "Monitor Server Dengan Prometheus dan Grafana"
date: "2023-05-26"
tags: ["Node Exporter", "Prometheus", "Grafana"]
categories: ["Monitoring"]
ShowToc: true
TocOpen: true
---

# Topologi


![](/images/diagram-prometheus.png)

# Install Node Exporter

Node exporter adalah aplikasi yang digunakan untuk mengekspor metrik dari sebuah sistem atau node ke dalam format yang dapat digunakan oleh sistem monitoring seperti prometheus. Node exporter berfungsi sebagai agen yang diinstall pada setiap node atau VM yang ingin di monitoring.

Buat user node_exporter untuk menjalankan service node exporter.

```bash
useradd --no-create-home --shell /bin/false node_exporter
```

Download dan install packages node exporter.

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar xvf node_exporter-1.5.0.linux-amd64
cd node_exporter-1.5.0.linux-amd64/
chown node_exporter:node_exporter node_exporter
cp node_exporter /usr/local/bin/
```

Untuk melihat collectors node exporter jalankan.

```bash
root@lab-node-exporter-1:~# node_exporter
```

Buat service node exporter.

```bash
nano /etc/systemd/system/node_exporter.service
#...
[Unit]
Description=Node Exporter

[Service]
User=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter --collector.systemd --collector.cpu --collector.meminfo --collector.loadavg --collector.uname \
   --collector.stat --collector.filesystem --collector.netstat
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
#...
```

Restart service node exporter.

```bash
systemd daemon-reload
systemctl restart node_exporter.service
systemctl enable node_exporter.service
systemctl status node_exporter.service
```

Cek listen port dari node exporter.

```bash
ss -tunlp |grep 9100
```

Check metrics node exporter.

```bash
curl http://<ip-node-exporter>:9100/metrics
```

# Install Prometheus

Prometheus adalah sistem monitoring open source yang dirancang khusus untuk memantau dan mengumpulkan data metrik dari berbagai sistem dan layanan.

Buat user prometheus untuk menjalankan service prometheus.

```bash
useradd --no-create-home --shell /bin/false prometheus
```

Download dan installl packages prometheus.

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.44.0/prometheus-2.44.0.linux-amd64.tar.gz
tar xvf prometheus-2.44.0.linux-amd64.tar.gz
cd prometheus-2.44.0.linux-amd64/
chown -r prometheus:prometheus prometheus promtool console_libraries consoles prometheus.yml
cp prometheus promtool /usr/local/bin
mkdir /var/lib/prometheus
mkdir /etc/prometheus
cp -r console_libraries console prometheus.yml /etc/prometheus/
```

Edit file konfigurasi prometheus.

```bash
nano /etc/prometheus/prometheus.yml

#...
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  scrape_timeout: 10s

scrape_configs:
  - job_name: "prometheus"
    metrics_path: /metrics
    scheme: http
    honor_timestamps: true
    static_configs:
      - targets: ["172.18.251.30:9090"]

  - job_name: "node-exporter"
    metrics_path: /metrics
    scheme: http
    honor_timestamps: true
    static_configs:
      - targets:
        - 172.18.251.11:9100
        - 172.18.251.12:9100
#...
```

Buat service prometheus.

```bash
nano /etc/systemd/system/prometheus/prometheus.service
#...
[Unit]
Description=Prometheus Server

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ --web.external-url=http://172.18.251.30:9090 --storage.tsdb.max-block-duration=2h \
    --storage.tsdb.min-block-duration=2h --storage.tsdb.retention.time=365d --storage.tsdb.wal-compression --web.enable-lifecycle \
    --web.enable-admin-api --log.level=debug --web.console.templates=/etc/prometheus/consoles
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
#...
```

Lakukan pengecekan apakah konfigurasi file prometheus sudah valid.

```bash
promtool check config /etc/prometheus/prometheus.yml
```

Restart service prometheus.

```bash
systemctl daemon-reload
systemctl restart prometheus.service
systemctl enable prometheus.service
systemctl status prometheus.service
```

Cek versi prometeheus.

```bash
promtool --version
```

Cek listen port prometheus.

```bash
ss -tunlp |grep 9090
```

Apakah ada jobs yang running.

```bash
promtool query instant 'http://<ip-prometheus>:9090'  'up==1'
```

Akses dashboard prometheus.

![](/images/dashboard-prometheus.png)

# Install Grafana

Grafana adalah software open source yang digunakan untuk membuat visualisasi atau dashboard monitoring dari metrics yang sudah di kumpulkan oleh prometheus.

Install dependencies packages grafana.

```bash
apt update
apt-get install -y gnupg2 curl software-properties-common
```

Tambahkan repository.

```bash
curl https://packages.grafana.com/gpg.key | sudo apt-key add -
add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
apt update
```

Install grafana.

```bash
apt-cache policy grafana
apt -y install grafana=9.5.2
```

Restart service grafana.

```bash
systemctl enable grafana-server.service
systemctl restart grafana-server.service
systemctl status grafana-server.service
```

Cek versi grafana.

```bash
grafana-cli --version
```

Cek listen port grafana.

```bash
ss -tunlp |grep 3000
```

Akses dashboard grafana.

![](/images/dashboard-grafana.png)