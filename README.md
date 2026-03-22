# nouci
**N**et**O**ps **U**tility **C**loud-**I**nit

nouci is a *low-toil* "batteries included" network monitoring widget, deployeable to any IaaS that supports Ubuntu LTS and [cloud-init](https://cloud-init.io/).

<p align="center">
    <img src="https://raw.githubusercontent.com/dlotterman/nouci/refs/heads/main/docs/nouci.png" alt="nouci diagram" width="750">
  </a>
</p>

With minimal operator configuration or maintenance, nouci turns a commodity cloud instance to a value packed monitoring tool:
- **Latency** trending (HTTP & ICMP) via [blackbox_exporter](https://github.com/prometheus/blackbox_exporter)
  - Pre-configured to monitor well known or "named" public endpoints
    - Hyperscalers and Compute providers
    - DNS and CDN endpoints
    - Assorted high value endpoints
- **SNMP** trending via [snmp_exporter](https://github.com/prometheus/blackbox_exporter)
- **Netflow**/**IPFIX**/**sFlow** collection via [akvorado](https://github.com/akvorado/akvorado)
- **Server** monitoring via [node_exporter](https://github.com/prometheus/node_exporter) collection
- **Syslog** via [graylog](https://go2docs.graylog.org/current/home.htm)
- **Graphing** via [Grafana](https://grafana.com/oss/)
- **Backups** via [restic + s3](https://restic.readthedocs.io/en/stable/)
