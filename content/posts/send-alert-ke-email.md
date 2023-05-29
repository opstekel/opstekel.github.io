---
title: "Mengirim Notifikasi Alert Grafana ke Email"
description: "Mengirim Notifikasi ke Email Menggunakan SMTP"
date: "2023-05-28"
tags: ["Grafana", "AlertManager"]
categories: ["Monitoring"]
ShowToc: true
TocOpen: true
---

# Prasyarat

- Sudah terinstall prometheus dan grafana.

Baca juga:

[Install Prometheus dan Grafana](https://blog.opstekel.com/posts/install-prometheus-grafana/)

[Install Alertmanager](https://blog.opstekel.com/posts/install-alertmanager/)

# Setting Contact Point

Setting contact point di dashboard grafana. Email address yang di isi pada contact point merupakan email address tujuan.

![](/images/email11.png)

# Setting SMTP

Tambahkan SMTP section pada konfigurasi file grafana.

```bash
nano /etc/grafana/grafana.ini
#...
# Setting di bawah section [smtp]
enabled = true
host = smtp.gmail.com:587
user = your-email@gmail.com
password = kdnjfoctswrpyanx
from_address = your-email@gmail.com
from_name = ginton
#...
```

> Catatan:
> 
> host = mengirim notifikasi ke email menggunakan port 587
>
> user = merupakan email address pengirim
>
> password = merupakan password user email your-email@gmail.com dari setelah generate app password di settingan gmail

Generate password.

Account -> Manage your Google Account.

![](/images/email12.png)

Security.

![](/images/email5.png)

2-Step Verification and Scroll Down.

![](/images/email6.png)

App passwords.

![](/images/email7.png)

select app: Other (Custom name).

![](/images/email8.png)

Generate.

![](/images/email9.png)

Copy password dan paste di section password [smtp].

![](/images/email10.png)

Restart service grafana.

```bash
systemctl restart grafana-server.service
```

# Verifikasi

Send test notification apakah sudah berhasil atau belum.

![](/images/email15.png)