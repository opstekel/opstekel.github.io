---
title: "Install AlertManager"
description: "Mengirim Notifikasi dengan AlertManager"
date: "2023-05-27"
tags: ["AlertManager"]
categories: ["Monitoring"]
ShowToc: true
TocOpen: true
---

# Pendahuluan

AlertManager adalah salah satu komponen utama dalam sistem pemantauan prometheus. Prometheus digunakan untuk mengumpulkan dan menyimpan metrik real time dari sistem yang sedang dimonitoring, sementara alertmanager bertanggung jawab untuk mengelola dan mengirimkan notifikasi atau peringatan (alert) berdasarkan kondisi tertentu.

AlertManager bisa di install di node terpisah dengan prometheus dan grafana, namun pada contoh ini alertmanager di install di node prometheus dan grafana. Sebelum melanjutkan lebih baik membaca terlebih dahulu [install-prometheus-grafana](https://blog.opstekel.com/posts/prometheus-grafana/)

# Install AlertManager

Download dan install alertmanager.

```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz
tar xvf alertmanager-0.25.0.linux-amd64.tar.gz
cd alertmanager-0.25.0.linux-amd64/
mkdir /etc/alertmanager
cp alertmanager.yml /etc/alertmanager/
cp alertmanager amtool /usr/local/bin/
```

Buat service alertmanager.

```bash
nano /etc/systemd/system/alertmanager.service
#...
[Unit]
Description=AlertManager Server Service
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
Type=simple
ExecStart=/usr/local/bin/alertmanager --config.file /etc/alertmanager/alertmanager.yml --web.external-url=http://<ip-node-alertmanager>:9093


[Install]
WantedBy=multi-user.target
#...
```

Edit file konfigurasi alertmanager

```bash
nano /etc/alertmanager/alertmanager.yml
#...
global:
  resolve_timeout: 5m

route:
 group_by: ['alertname']
 receiver: slack_general
 routes:
  - match:
      severity: slack
    receiver: slack_general

receivers:
- name: slack_general
  slack_configs:
  # Setting slack agar enable incoming webhooks dan copy url ke bawah ini (optional).
  - api_url: 'https://hooks.slack.com/services/<incoming-web-hooks-service>'
    channel: '#alert'
    icon_url: https://avatars3.githubusercontent.com/u/3380462
    title: |-
      [{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .CommonLabels.alertname }} for {{ .CommonLabels.job }}
      {{- if gt (len .CommonLabels) (len .GroupLabels) -}}
        {{" "}}(
        {{- with .CommonLabels.Remove .GroupLabels.Names }}
          {{- range $index, $label := .SortedPairs -}}
            {{ if $index }}, {{ end }}
            {{- $label.Name }}="{{ $label.Value -}}"
          {{- end }}
        {{- end -}}
        )
      {{- end }}
    text: >-
      {{ range .Alerts -}}
      *Alert:* {{ .Annotations.title }}{{ if .Labels.severity }} - `{{ .Labels.severity }}`{{ end }}

      *Description:* {{ .Annotations.description }}

      *Details:*
        {{ range .Labels.SortedPairs }} • *{{ .Name }}:* `{{ .Value }}`
        {{ end }}
      {{ end }}

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
#...
```

Restart service alertmanager.

```bash
systemctl daemon-reload
systemctl restart alertmanager.service
systemctl enable alertmanager.service
systemctl status alertmanager.service
```

Cek listen port alertmanager

```bash
ss -tunlp |grep 9093
```

Akses dashboard alertmanager, ip-alertmanager:9093. 

![](/images/alertmanager.png)

# Integrasi alertmanager dengan prometheus.

Kita perlu menambahkan alerting di konfigurasi prometheus.

Di node prometheus, tambahkan:

```bash
nano /etc/prometheus/prometheus.yml
#...
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  scrape_timeout: 10s
# Tambahkan line di bawah ini
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 172.18.251.30:9093
#...
```

Restart service prometheus.

```bash
systemctl restart prometheus.service
```