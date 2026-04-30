# nouci
**N**et**O**ps **U**tility **C**loud-**I**nit

nouci is a *low-toil* "batteries included" network monitoring widget, deployeable to any IaaS / BMaaS that supports Ubuntu LTS and [cloud-init](https://cloud-init.io/).

While it **is** intended to be immediately useful as a quick "run + copy + paste + launch" paste tool for operators in firefights who need quick views / POPs, it is mostly intended to be a reference or lab environment utility.

* This project is still WIP, expect frequent changes.

<p align="center">
    <img src="https://raw.githubusercontent.com/dlotterman/nouci/refs/heads/main/docs/assets/grafana_dash.png" alt="nouci diagram" width="750">
  </a>
</p>

<p align="center">
    <img src="https://raw.githubusercontent.com/dlotterman/nouci/refs/heads/main/docs/assets/nouci.png" alt="nouci diagram" width="750">
  </a>
</p>

Please see our [docs](docs/):
* [Getting Started with Libvirt](docs/getting_started_libvirt.md)
* [Getting Started with Latitude.sh](docs/getting_started_latitude.md)

With minimal operator configuration or maintenance, nouci turns a commodity cloud instance to a value packed monitoring tool:
- **Latency** trending (ICMP, HTTP and DNS) via [blackbox_exporter](https://github.com/prometheus/blackbox_exporter)
  - Pre-configured to monitor well known or "named" public endpoints
    - Hyperscalers and Compute providers
    - DNS and CDN endpoints
    - Assorted high value endpoints
- **SNMP** trending via [snmp_exporter](https://github.com/prometheus/snmp_exporter)
- **Server** monitoring via [node_exporter](https://github.com/prometheus/node_exporter)
- **Netflow**/**IPFIX**/**sFlow** planned
- **Syslog planned
- **Graphing** via [Grafana](https://grafana.com/oss/)
- **VNF Hosting** via [libvirt](https://libvirt.org/) (aka "KVM")
	- Also exposed with UI for management via Cockpit
- **Backups** planned
- **HTTPS** planned
- **IPsec module** planned
- **Wireguard module** planned
- **Disk Encryption** planed