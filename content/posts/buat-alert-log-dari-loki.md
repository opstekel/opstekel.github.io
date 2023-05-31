---
title: "Membuat Alerting Grafana Dari Log Loki"
description: "Membuat Alert Ketika Ada Log Anomali Dengan Loki Sebagai DataSource"
date: "2023-05-31"
tags: ["Promtail", "Loki", "Grafana", "AlertManager"]
categories: ["Monitoring", "Logs"]
ShowToc: true
TocOpen: true
---

# Prasyarat

- Sudah install promtail, loki, grafana. Link [install-promtail-dan-loki](https://blog.opstekel.com/posts/grafana-loki/)
- Sudah install alertmanager. Link [install-alertmanager](https://blog.opstekel.com/posts/install-alertmanager/)

# Integrasi AlertManager dengan Loki

Untuk konfigurasi loki lengkapnya dan install alert manager bisa merujuk pada link di atas.

Edit file konfigurasi loki.

```bash
nano /etc/loki/local-config.yaml
#...
server:
  http_listen_port: 3100
# Tambahkan line dibawah ini
ruler:
  storage:
    type: local
    local:
      directory: /data_loki/rules
  rule_path: /tmp/rules/fake/
  alertmanager_url: http://<ip-alert-manager>:9093
  ring:
    kvstore:
      store: inmemory
  enable_api: true
#...
```

Restart service loki.

```bash
systemctl restart loki.service
systemctl status loki.service
```

# Membuat Alert Dari Log

Saat ini kita ingin jika ada log error pada sistem **/var/log/syslog** ada notifikasi alert. Kita bisa menentukan isi log apa yang ingin dijadikan alert, namun pada contoh ini saya mengambil contoh jika ada log error.

- Explore log terlebih dahulu

![](/images/alert-grafana1.png)

Kita ingin mengambil log error dengan case insensitive.

- Membuat dashboard

![](/images/alert-grafana2.png)

Perlu diperhatikan bahwa data source yang digunakan adalah loki dan disini kita menggunakan **count_over_time**, kenapa? karena alert rule tidak bisa memproses output yang sangat panjang.

- Membuat rule alert

Kita akan menghitung berapa jumlah log error di syslog, jika lebih dari 0 atau ada error maka kita akan set untuk mengirim alert.

Buat lah alert rule seperti di bawah.

![](/images/alert-grafana3.png)

![](/images/alert-grafana4.png)

![](/images/alert-grafana5.png)

Labels berfungsi untuk memberi tanda pada alert rule.

- Membuat notification policy

**Notification policies** -> **+ New nested policy**

![](/images/alert-grafana6.png)

Setting label sesuai pada alert rule yang sudah kita buat sebelumnya. Dan contact point merupakan tujuan alert ini dikirim.

# Verifikasi

Test input **error** pada syslog di instance yang sedang dimonitoring lognya. Dan cek status alert rulenya apakah berubah menjadi pending lalu firing.

```bash
echo "Error" >> /var/log/syslog
```