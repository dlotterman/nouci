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

- There are a lot of hard coded magic strings and paths. This is tricky because we are limited in variable execution cause we are gluieind across so many widgets, many do not pass variables to each other, and we don't want to build ansible.
	- For anything not secret, abuse of sourcing /etc/defaults client side could help
	- Could build a templater into the buildchain
	- Current places this is most painful are snmp and grafana which have hardcoded paths to self-checkout git, which is exactly why you try to avoid this problem.

- When re-do local-disk / disk-enc, should use systemd mount units instead of mangling fstab / crypt