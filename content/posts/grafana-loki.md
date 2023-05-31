---
title: "Install Promtail dan Loki"
description: "Monitoring Log Sistem Menggunakan Promtail dan Loki"
date: "2023-05-30"
tags: ["Promtail", "Loki", "Grafana"]
categories: ["Monitoring", "Logs"]
ShowToc: true
TocOpen: true
---

# Topologi

![](/images/grafana-loki1.png)

# Penjelasan

**Promtail**: Agen pengumpul log dari berbagai sumber seperti file teks, syslog, dan jurnal sistem kemudian mengirimkannya ke penyimpanan log yang disebut loki. Install promtail di setiap node atau vm yang ingin dipantau.

**Loki**: Penyimpanan log yang dirancang untuk menyimpan dan mengindeks log secara efisien. Loki menggunakan struktur penyimpanan berbasis waktu yang disebut "index" dan "chunk". Log yang dikumpulkan oleh promtail dikirim ke loki untuk disimpan.

**Grafana**: berfungsi sebagai visualisasi jika ingin memonitoring jumlah log ataupun ingin menggunakan fitur grafana alert agar ketika ada log error maka akan mengirimkan notifikasi (optional). Baca juga artikel [mengirimkan notifikasi error log](https://blog.opstekel.com/posts/buat-alert-log-dari-loki/).


# Install Promtail

Download package promtail. Lakukan di semua node yang ingin dipantau.

```bash
curl -s https://api.github.com/repos/grafana/loki/releases/latest | grep browser_download_url |  cut -d '"' -f 4 | grep promtail-linux-amd64.zip | wget -i -
unzip promtail-linux-amd64.zip
mv promtail-linux-amd64 /usr/local/bin/promtail
mkdir /etc/promtail
```

Buat file konfigurasi promtail, Kita akan mengambil log di semua file yang mempunyai nama akhiran log.

```bash
nano /etc/promtail/local-config.yaml
#...
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://172.18.251.30:3100/loki/api/v1/push

scrape_configs:
- job_name: oslog
  static_configs:
  - targets:
      - localhost
    labels:
      host: server1
      __path__: /var/log/*log
#...
```

Buat service promtail.

```bash
nano /etc/systemd/system/promtail.service
#...
[Unit]
Description=Promtail service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/promtail -config.file /etc/promtail/local-config.yaml

[Install]
WantedBy=multi-user.target
#...
```

Restart service promtail.

```bash
systemctl daemon-reload
systemctl enable promtail.service
systemctl restart promtail.service
systemctl status promtail.service
```

Cek versi promtail.

```bash
promtail --version
```

Cek listen port promtail.

```bash
ss -tunlp |grep 9080
```

# Install Loki

Download package loki.

```bash
curl -s https://api.github.com/repos/grafana/loki/releases/latest | grep browser_download_url |  cut -d '"' -f 4 | grep loki-linux-amd64.zip | wget -i -
unzip loki-linux-amd64.zip
mv loki-linux-amd64 /usr/local/bin/loki
mkdir /data_loki/loki
mkdir /etc/loki
```

Buat file konfigurasi loki.

```bash
nano /etc/loki/local-config.yaml
#...
auth_enabled: false

server:
  http_listen_port: 3100

ingester:
  lifecycler:
    address: 172.18.251.30
    ring:
      kvstore:
        store: inmemory
      replication_factor: 1
    final_sleep: 0s
  chunk_idle_period: 5m
  chunk_retain_period: 30s
  max_transfer_retries: 0

schema_config:
  configs:
    - from: 2023-05-30
      store: boltdb
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 168h

storage_config:
  boltdb:
    directory: /data_loki/loki/index

  filesystem:
    directory: /data_loki/loki/chunks

limits_config:
  enforce_metric_name: false
  reject_old_samples: true
  reject_old_samples_max_age: 168h
  ingestion_rate_mb: 30
  ingestion_burst_size_mb: 30
  max_entries_limit_per_query: 50000000000

chunk_store_config:
  max_look_back_period: 0s

table_manager:
  retention_deletes_enabled: false
  retention_period: 0s
#...
```

Buat service loki.

```bash
nano /etc/systemd/system/loki.service
#...
[Unit]
Description=Loki service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/loki -config.file /etc/loki/local-config.yaml

[Install]
WantedBy=multi-user.target
#...
```

Restart service loki.

```bash
systemctl daemon-reload
systemctl restart loki.service
systemctl enable loki.service
systemctl status loki.service
```

Cek versi loki.

```bash
loki --version
```

Cek listen port loki.
```bash
ss -tunlp |grep 3100
```

Untuk install grafana bisa lihat di artikel berikut [install-grafana](https://blog.opstekel.com/posts/install-prometheus-grafana/#install-grafana).