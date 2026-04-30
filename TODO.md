# General Todo
- Consolidate network configuration
	- Currently have the dummy interface being created by `systemd-networkd` in `base`
  - Currently have the fake0 interface being created by `nmcli` to help with the cockpit bug
  - Currently have a `systemd-networkd-wait-online` drop-in

- Consolidate `/mnt` and some `/opt` directories

- Figure out a way to not inception check out the repo to get at grafana assets

- Need to put everything behind SSL, even if self signed.

- Fix blackbox CDNs to be forgiving of `4xx`
- Add additional near-hop discovery
- Add more interesting endpoints to blackbox default

- Provide snmp dashboard