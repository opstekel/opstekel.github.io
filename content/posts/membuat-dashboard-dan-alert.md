---
title: "Membuat Dashboard dan Mengirim Notifikasi ke Telegram"
description: "Add Data Source, Create Panel Dashboard, Create Alert and Send to Telegram"
date: "2023-05-28"
tags: ["Grafana"]
categories: ["Monitoring"]
ShowToc: true
TocOpen: true
---

# Add Datasource

Toggle Menu -> Administrations -> Data Sources -> Add new data source -> prometheus.

![](/images/datasource1.png)

![](/images/datasource2.png)

![](/images/datasource3.png)

Tambahkan data source prometheus dengan IP node prometheus.

![](/images/datasource4.png)

# Create Dashboard

Toggle Menu -> Dashboards -> Go To Folder -> Create Dashboard -> Add Visualization 

![](/images/create-dashboard2.png)

![](/images/create-dashboard3.png)

![](/images/create-dashboard4.png)

Edit dashboard seperti gambar dibawah (total node yang aktif) lalu save.

![](/images/create-dashboard6.png)

# Create Alert

Membuat alert ketika ada node yang tidak aktif maka akan mengirim notifikasi ke telegram.

Step 1: Create alert pada panel dashboard.

![](/images/alert1.png)

Buat rule **B** untuk mengambil nilai terakhir dari rule **A**. Kemudian rule **C** untuk menentukan jika nilai dari node yang up (rule **B**) kurang dari 2 maka ada node down yang akan trigger alert untuk mengirimkan notifikasi.

![](/images/alert2.png)

**evaluation interval** berfungsi untuk melakukan pengecekan aturan rule secara berkala apakah memenuhi kondisi atau tidak. Jika diatur evaluation interval ke 1 menit, grafana akan memeriksa kondisi alert setiap menit. 

**for 2m** adalah untuk menentukan berapa lama suatu kondisi harus dipertahankan sebelum alert dikirimkan, Jika diatur for selama 2m maka kondisi nilai akan dipertahankan selama 2 menit jika benar selama 2 menit kondisi nya memenuhi kondisi alert maka akan mengirim notifikasi ke telegram. **for** digunakan untuk menghilangkan pemberitahuan palsu atau flapping (pemberitahuan yang berulang kali dikirimkan secara berulang dalam periode waktu yang singkat) yang dapat terjadi ketika metrik melonjak naik dan turun dengan cepat.

**labels** juga penting untuk **notification policies** tau bahwa alert rule mana akan dikirimkan kemana.

![](/images/alert3.png)

Step 2: Buat **contact points** ke telegram. Isikan token bot API telegram dan chat ID telegram.

Note: untuk mencari chat ID telegram bisa dengan: https://api.telegram.org/bot(token)/getUpdates

![](/images/alert4.png)

![](/images/alert5.png)

Step 3: Buat **notification policies**, isi label sesuai dengan label di **alert rule** dan pilih **contact points** yang sudah kita buat.

![](/images/alert6.png)

![](/images/alert7.png)

## Verifikasi

Matikan salah satu instance atau vm node exporter.

```bash
shutdown -h now
```

Periksa kembali 1 sampai 2 menit status alert rule akan berubah menjadi pending kemudian firing.

![](/images/alert8.png)

![](/images/alert10.png)

Kemudian mengirimkan notifikasi ke telegram.

![](/images/alert9.png)