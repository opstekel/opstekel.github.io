---
title: "Install Libvirt Exporter"
description: "Melakukan Monitoring VM yang Berjalan Di Atas Libvirt"
date: "2023-05-31"
tags: ["Prometheus", "Libvirt Exporter"]
categories: ["Monitoring"]
ShowToc: true
TocOpen: true
---

# Pendahuluan

Libvirt expoter adalah sebuah software yang dirancang untuk mengumpulkan dan mengekspor data metrics dari virtualisasi yang menggunakan teknologi libvirt ke prometheus.

# Install Libvirt Exporter

Install snapd.

```bash
apt update -y && apt install snapd -y
```

Install libvirt exporter.

```bash
snap install prometheus-libvirt-exporter
systemctl status snap.prometheus-libvirt-exporter.daemon.service
snap connect prometheus-libvirt-exporter:libvirt
```

Restart service libvirt expoter.

```bash
systemctl restart snap.prometheus-libvirt-exporter.daemon.service
systemctl status snap.prometheus-libvirt-exporter.daemon.service
```

Cek listen port libvirt expoter.

```bash
ss -tunlp |grep 9177
```

Cek metric libvirt expoter.

```bash
curl <ip-libvirt-expoter>:9177/metrics
```

# Integrasi Libvirt Expoter dengan Prometheus

Jika sudah install prometheus, tambahkan job pada file konfigurasi prometheus.

```bash
#...
scrape_configs:
# Tambahkan line dibawah ini
  - job_name: "libvirt-exporter"
    metrics_path: /metrics
    scheme: http
    honor_timestamps: true
    static_configs:
      - targets: ["<ip-libvirt-exporter>:9177"]
#...
```

# Verifikasi

Akses dashboard prometheus dan lihat apakah ada metrics libvirt expoter.

![](/images/libvirt-exporter.png)

Link [install-prometheus](https://blog.opstekel.com/posts/install-prometheus-grafana/#install-prometheus)