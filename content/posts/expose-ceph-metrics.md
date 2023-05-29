---
title: "Expose Ceph Metrics"
description: "Mengekspos Metrik Ceph Cluster"
date: "2023-05-29"
tags: ["Ceph Metrics", "Prometheus"]
categories: ["Ceph Storage"]
ShowToc: true
TocOpen: true
---

# Prasyarat

- Mempunyai Ceph Cluster


# Enable Plugin Prometheus

Enable plugin prometheus di node cluster ceph, lakukan di salah satu ceph-mon node nya saja.

```bash
ceph mgr module enable prometheus --force
```

Lakukan pengecekan expose ceph metric di IP dan Port berapa.

```bash
ceph mgr services
```

Cek listen port ceph exporter.

```bash
ss -tunlp |grep 9283
```

Cek metrics.

```bash
curl http://ip-ceph:9283/metrics
```

# Integrasi Ceph Exporter dengan Prometheus

Tambahkan job ceph exporter di prometheus.

```bash
nano /etc/prometheus/prometheus.yml
#...
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  scrape_timeout: 10s
scrape_configs:
# Tambahkan line dibawah ini
    - job_name: "ceph-exporter"
        scrape_interval: 15s
        honor_timestamps: true
        scrape_timeout: 10s
        scheme: http
        metrics_path: /metrics
        static_configs:
          - targets:
            - ip-ceph1:9283
            - ip-ceph2:9283
            - ip-ceph3:9283
            labels:
              alias: ceph-exporter
#...
```

> Note:
>
> Jika menggunakan 3 ceph monitor maka masukkan saja ketiga ip ceph mon tersebut ke dalam konfigurasi prometheus, karena yang menjadi leader dari ceph mon tidak tetap di satu ceph mon node saja (berpindah-pindah).

Lakukan pengecekan apakah metric ceph sudah bisa di query dari prometheus dashboard.

![](/images/ceph-metric.png)

# Custom IP dan Port Ceph Exporter

```bash
ceph config set mgr mgr/prometheus/server_addr <ip-address>
ceph config set mgr mgr/prometheus/server_port <port>
```

Contoh:

```bash
ceph config set mgr mgr/prometheus/server_addr 0.0.0.0
ceph config set mgr mgr/prometheus/server_port 9283
```