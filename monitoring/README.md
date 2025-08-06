## Prometheus PVE Exporter

[See Prometheus PVE Exporter Github Page](https://github.com/prometheus-pve/prometheus-pve-exporter)

**1. Create a PVEAuditor user**

Use PVE auth, **not PAM**

**2. Use password auth**

Since token auth was not working, export this config as a volume at `/etc/prometheus/pve.yml` on the node exporter host:

```yaml
# /etc/prometheus/pve.yml
default:
    user: monitoring@pve
    password: sEcr3T!
    # Optional: set to false to skip SSL/TLS verification
    verify_ssl: false
```

Visit `http://localhost:9221/pve?target=192.168.2.100&cluster=1&node=1` where `192.168.2.100` is the IP of the Proxmox VE node to get metrics from. 

**3. Prometheus Exporter Config**

[See Prometheus Install Docs](https://prometheus.io/docs/prometheus/latest/installation/#using-docker)

Export this config as a volume at `/etc/prometheus/prometheus.yml` on your Prometheus host.

```yaml
# /etc/prometheus/prometheus.yml
scrape_configs:
  - job_name: 'pve'
    static_configs:
      - targets:
        - 192.168.2.100  # Proxmox VE node.
    metrics_path: /pve
    params:
      module: [default]
      cluster: ['1']
      node: ['1']
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 10.0.0.202:9221  # PVE exporter.
```

**4. Grafana Dashboard**

[See Grafana Install Docs](https://grafana.com/docs/grafana/latest/setup-grafana/installation/docker/)

[You can use this dashboard](https://grafana.com/grafana/dashboards/10347-proxmox-via-prometheus/) to visualize your PVE metrics

## Uptime Kuma Grafana Dashboard

[Refer to this article](https://tomerklein.dev/real-time-uptime-monitoring-with-uptime-kuma-and-grafana-16638d6a579f)

**1. Create API Key**

`Home > Settings > API Key`

You can see uptime-kuma metrics via the `/metrics/` endpoint

**2. Configure Prometheus Scraper**

```yaml
  - job_name: 'uptime'
    scrape_interval: 30s
    scheme: http
    static_configs:
      - targets: ['uptime.example.com']
    basic_auth: 
      password: <API_KEY>
```

**3. Import Uptime-kuma Grafana Dashboard**

[Uptime Kuma — SLA/Latency/Certs](https://grafana.com/grafana/dashboards/18667-uptime-kuma-metrics/) <-- currently in use

[Uptime Kuma - Metrics](https://grafana.com/grafana/dashboards/18278-uptime-kuma/)

## Manage Docker Compose Containers

[See Dockge](https://github.com/louislam/dockge)
