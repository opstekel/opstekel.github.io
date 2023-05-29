---
title: "Mengirim Notifikasi Alert Grafana ke Slack"
description: "Mengirim Notifikasi ke Multiple Channel di Slack"
date: "2023-05-29"
tags: ["Grafana", "AlertManager"]
categories: ["Monitoring"]
ShowToc: true
TocOpen: true
---

# Prasyarat

- Sudah terinstall prometheus dan grafana.

- Mempunyai akun slack.

Baca juga:

[Install Prometheus dan Grafana](https://blog.opstekel.com/posts/install-prometheus-grafana/)

[Install Alertmanager](https://blog.opstekel.com/posts/install-alertmanager/)

# Setting Slack

Klik workspace name.

![](/images/slack2.png)

**Settings & administration -> Manage apps**.

![](/images/slack3.png)

Search **Incoming WebHooks**.

![](/images/slack4.png)

**Add to slack**.

![](/images/slack5.png)

Pilih channel -> **Add Incoming WebHooks integration**.

> Note:
>
> channel disini hanya pilihan saja, nanti kita tetap bisa menentukan tujuan channel di slack menggunakan grafana.

![](/images/slack6.png)

Copy *Webhook URL*

![](/images/slack7.png)

# Setting Grafana.

Disini kita akan mencoba mengirim notifikasi alert ke beberapa channel di slack menggunakan 1 webhook url.

Login Dashboard grafana.

**Contact points** -> **Add contact point**

![](/images/slack9.png)

Set Integration ke slack -> set channel pada bagian **Recipient** -> paste webhook URL -> kemudian test kirim.

![](/images/slack11.png)

Alert firing masuk ke channel yang kita setting.

![](/images/slack12.png)

Coba lagi test kirim notifikasi dengan webhook url yang sama tapi dengan channel yang berbeda.

![](/images/slack13.png)

Maka alert firing masuk ke channel yang kita setting.

![](/images/slack14.png)


