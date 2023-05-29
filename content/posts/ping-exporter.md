---
title: "Install Ping Exporter"
description: "Melakukan Monitoring Koneksi Instance Menggunakan Ping Exporter"
date: "2023-05-29"
tags: ["Ping Exporter"]
categories: ["Monitoring"]
ShowToc: true
TocOpen: true
---

# Topologi

![](/images/ping_exporter1.png)

# Pendahuluan

Ping exporter adalah aplikasi yang digunakan untuk mengukur kualitas jaringan dan mengumpulkan statistik ping dari berbagai host atau perangkat. Ping exporter biasanya digunakan untuk melakukan monitoring konektifitas pada jaringan apakah terjadi ping loss (putus koneksi) ataupun untuk memonitor latency jaringan.

# Install Ping Exporter

Download package ping exporter di node ping exporter (172.18.251.12).

```bash
wget https://github.com/czerwonk/ping_exporter/releases/download/1.0.1/ping_exporter_1.0.1_linux_amd64.tar.gz
tar xvf ping_exporter_1.0.1_linux_amd64.tar.gz
cp ping_exporter /usr/local/bin/
```

Buat file konfigurasi ping exporter.

```bash
mkdir /etc/ping_exporter/config.yml
#...
targets:
  - 172.18.251.11

ping:
  interval: 2s
  timeout: 3s
  history-size: 42
  size: 120
#...
```

Buat service ping exporter.

```bash
nano /etc/systemd/system/ping_exporter.service
#...
[Unit]
Description=Ping Exporter
After=network.target

[Service]
User=root
ExecStart=/usr/local/bin/ping_exporter --config.path=/etc/ping_exporter/config.yml
NoNewPrivileges=yes
CapabilityBoundingSet=CAP_NET_RAW
AmbientCapabilities=CAP_NET_RAW
PrivateDevices=true
PrivateTmp=yes
ProtectControlGroups=true
ProtectKernelModules=yes
ProtectKernelTunables=true
ProtectSystem=strict
ProtectClock=true
ProtectHostname=true
ProtectHome=true
DevicePolicy=closed
RestrictNamespaces=yes
RestrictRealtime=yes
RestrictSUIDSGID=yes
MemoryDenyWriteExecute=yes
LockPersonality=yes
Restart=always
RestartSec=3

[Install]
WantedBy=default.target
#...
```

Restart service ping exporter.

```bash
systemctl daemon-reload
systemctl restart ping_exporter.service
systemctl enable ping_exporter.service
systemctl status ping_exporter.service
```

Cek listen port ping exporter.

```bash
ss -tunlp |grep 9427
```

Cek metric-metric ping exporter.

```bash
curl 172.18.251.12:9427/metrics
```

# Integrasi ping exporter dengan prometheus

Tambahkan job ping exporter di node prometheus.

```bash
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  scrape_timeout: 10s
scrape_configs:
## Tambahkan line di bawah ini.
  - job_name: "ping-exporter"
    static_configs:
      - targets:
        - 172.18.251.12:9427
```

Restart service prometheus.

```bash
systemctl restart prometheus.service
```

# Verifikasi

Lakukan pengecekan pada dashboard prometheus apakah metric ping exporter sudah masuk ke prometheus.

![](/images/ping_exporter2.png)