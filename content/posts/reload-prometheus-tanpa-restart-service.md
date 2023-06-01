---
title: "Reload Konfigurasi Prometheus Tanpa Restart Service"
description: "Melakukan Perubahan Pada File Konfigurasi Prometheus Tanpa Restart Service"
date: "2023-06-01"
tags: ["Prometheus"]
categories: ["Monitoring"]
ShowToc: true
TocOpen: true
---

# Pendahuluan

Biasanya setiap kali kita merubah file konfigurasi dari sebuah service atau aplikasi, maka kita perlu juga untuk melakukan restart pada service tersebut sehingga menyebabkan downtime pada service tersebut selama service tersebut direstart.

Dengan prometheus kita bisa melakukan perubahan pada file konfigurasi, misal menambahkan target tanpa restart service prometheus.

# Reload Konfigurasi Prometheus

Reload konfigurasi dengan mengirimkan HTTP POST ke prometheus web server.

```bash
curl -i -XPOST http://<ip-prometheus>:9090/-/reload
```

> NOTE:
>
> Perlu untuk menambahkan **--web.enable-lifecycle** pada file systemd prometheus.

Jika tidak menambahkan **--web.enable-lifecycle** maka output yang keluar:

```bash
HTTP/1.1 403 Forbidden
Date: Thu, 01 Jun 2023 02:26:14 GMT
Content-Length: 29
Content-Type: text/plain; charset=utf-8
```

Jika berhasil:

```bash
HTTP/1.1 200 OK
Date: Thu, 01 Jun 2023 02:27:40 GMT
Content-Length: 0
```

Kita juga bisa memvalidasi file konfigurasi prometheus apakah ada kesalahan syntax atau tidak dengan promtool [disini](https://blog.opstekel.com/posts/install-prometheus-grafana/#install-prometheus).