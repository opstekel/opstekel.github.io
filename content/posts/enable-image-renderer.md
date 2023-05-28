---
title: "Mengirim Notifikasi dengan Screenshot Image Dashboard"
description: "Enable Image Renderer Untuk Mengirim Screenshot Panel Dashboard ke Notifikasi"
date: "2023-05-28"
tags: ["Grafana", "AlertManager"]
categories: ["Monitoring"]
ShowToc: true
TocOpen: true
---

# Pendahuluan

Jika ingin notifikasi yang dikirim disertai dengan screenshot dari panel dashboard kita bisa mengaktifkannya dengan enable image renderer pada grafana. Perintah ini di jalankan di node grafana.

# Enable Image Renderer Plugin

Enable grafana image renderer plugin dengan menggunakan command.

```bash
grafana-cli plugins install grafana-image-renderer
```

Tambahkan section dibawah ke dalam file konfigurasi grafana.

```bash
nano /etc/grafana/grafana.ini
#...
# Tambahkan line dibawah ini
[unified_alerting.screenshots]
capture = true
##...
```

Install dependencies untuk menjalankan fungsi image renderer.

Ubuntu:

```bash
apt install libx11-6 libx11-xcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrender1 libxtst6 libglib2.0-0 libnss3 libcups2  libdbus-1-3 libxss1 libxrandr2 libgtk-3-0 libasound2 libxcb-dri3-0 libgbm1 libxshmfence1
```

Debian 9:

```bash
apt install libx11 libcairo libcairo2 libxtst6 libxcomposite1 libx11-xcb1 libxcursor1 libxdamage1 libnss3 libcups libcups2 libxss libxss1 libxrandr2 libasound2 libatk1.0-0 libatk-bridge2.0-0 libpangocairo-1.0-0 libgtk-3-0 libgbm1 libxshmfence1
```

Debian 10:

```bash
apt install libxdamage1 libxext6 libxi6 libxtst6 libnss3 libcups2 libxss1 libxrandr2 libasound2 libatk1.0-0 libatk-bridge2.0-0 libpangocairo-1.0-0 libpango-1.0-0 libcairo2 libatspi2.0-0 libgtk3.0-cil libgdk3.0-cil libx11-xcb-dev libgbm1 libxshmfence1
```

Centos 7:

```bash
apt install libXcomposite libXdamage libXtst cups libXScrnSaver pango atk adwaita-cursor-theme adwaita-icon-theme at at-spi2-atk at-spi2-core cairo-gobject colord-libs dconf desktop-file-utils ed emacs-filesystem gdk-pixbuf2 glib-networking gnutls gsettings-desktop-schemas gtk-update-icon-cache gtk3 hicolor-icon-theme jasper-libs json-glib libappindicator-gtk3 libdbusmenu libdbusmenu-gtk3 libepoxy liberation-fonts liberation-narrow-fonts liberation-sans-fonts liberation-serif-fonts libgusb libindicator-gtk3 libmodman libproxy libsoup libwayland-cursor libwayland-egl libxkbcommon m4 mailx nettle patch psmisc redhat-lsb-core redhat-lsb-submod-security rest spax time trousers xdg-utils xkeyboard-config alsa-lib
```

Centos 8:

```bash
apt install libXcomposite libXdamage libXtst cups libXScrnSaver pango atk adwaita-cursor-theme adwaita-icon-theme at at-spi2-atk at-spi2-core cairo-gobject colord-libs dconf desktop-file-utils ed emacs-filesystem gdk-pixbuf2 glib-networking gnutls gsettings-desktop-schemas gtk-update-icon-cache gtk3 hicolor-icon-theme jasper-libs json-glib libappindicator-gtk3 libdbusmenu libdbusmenu-gtk3 libepoxy liberation-fonts liberation-narrow-fonts liberation-sans-fonts liberation-serif-fonts libgusb libindicator-gtk3 libmodman libproxy libsoup libwayland-cursor libwayland-egl libxkbcommon m4 mailx nettle patch psmisc redhat-lsb-core redhat-lsb-submod-security rest spax time trousers xdg-utils xkeyboard-config alsa-lib libX11-xcb
```

Buat alert seperti referensi berikut [Create Alert](https://blog.opstekel.com/posts/membuat-dashboard-dan-alert/#create-alert)

Kemudian test shutdown node atau vm.

```bash
shutdown -h now
```

maka notifikasi juga akan mengirimkan screenshot gambar dari panel dashboard.

![](/images/image-renderer.png)