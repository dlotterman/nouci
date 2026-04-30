# Getting started with Latitude.sh

[Latitude.sh](https://www.latitude.sh/) (LSH) is a globally available Bare Metal Cloud platform which supports [cloud-init](https://cloud-init.io/).

Launching nouci with LSH should be straightforward.

You will need:
- A bash like shell
- The ability to install a recent [yq4](https://github.com/mikefarah/yq)
- An LSH account

## From your workstation shell

**Clone** this nouci repository and change directory to desired Ubuntu release, 2404 assumed:

```
git clone https://github.com/dlotterman/nouci && cd nouci/ubuntu_releases/2404
open init/nouci_init.yaml
```
* Substitute `open` for your editor of choice

**Edit** `nouci_init.yaml`:
- Provide / edit your Github or Launchpad username:

	`ssh_import_id: ["gh:dlotterman"]`
	- This imports SSH keys based on [cloud-init](https://docs.cloud-init.io/en/latest/reference/yaml_examples/ssh.html#import-ssh-id)'s use of [ssh-import-id](https://manpages.ubuntu.com/manpages/jammy/man1/ssh-import-id.1.html)
  - This is optional if you launch the instance with SSH keys from the Latitude.sh platform, which will get picked up organically
- Punch a firewall hole for your IP, edit or add:

	`    - [ufw, allow, from, 35.149.59.78, to, any, port, 22]`
	- Optionally, punch a hole for your IP to all services (some services only exposed over HTTP)

	` 	 - [ufw, allow, from, 35.149.59.78]`
	- Optionally, allow SSH in from anywhere (discouraged)

	` 	 - [ufw, allow, ssh]`
	- Optionally, provide a password to the `ubuntu` user (very discouraged)

	```
	password: NouciIsFun07
	chpasswd:
	    expire: false
	ssh_pwauth: true
	```

**Build** nouci:

Install `yq4`:

```
wget -O ~/.local/bin/yq4 https://github.com/mikefarah/yq/releases/download/v4.52.4/yq_linux_amd64
chmod +x ~/.local/bin/yq4
```

Use `yq4` to merge the multiple necessary yaml files together:

```
~/.local/bin/yq4 eval-all '. as $item ireduce ({}; . *+ $item )' modules/init/nouci_init.yaml \
modules/base/nouci_base.yaml \
modules/disk_local/nouci_disk_local_single.yaml  \
modules/disk_encryption/nouci_disk_local_single_enc.yaml \
modules/prometheus/nouci_prometheus.yaml \
modules/node_exporter/nouci_node_exporter.yaml \
modules/blackbox_exporter/nouci_blackbox_exporter.yaml \
modules/snmp_exporter/nouci_snmp_exporter.yaml \
modules/grafana/nouci_grafana.yaml | tee nouci.yaml
```

Congratulations! `nouci.yaml` is now your complete and ready to go **user-data** or `cloud-init` file.

## From the Latitude.sh Portal

1. [Follow the docs](https://www.latitude.sh/docs/servers/user-data) on adding user-data to project or team
	- Copy and Paste the entirely of the output / contents of `nouci.yaml` into the "Contents" sections of the form.
2. [Follow the docs](https://www.latitude.sh/docs/servers/deploying-a-server) on deploying a server
	- Deploy the version of Ubuntu chosen earlier (2404 was assumed)
	- Take special note to select your user-data from the previous step when launching an instance
	- The defaults should otherwise suffice unless a reason is known to pick another instance type
	- Name it something recognizable, ideal if forward DNS can be setup to match but not necessary
	- Include your SSH key if you like, this is complementary to the SSH keys you setup earlier in your `cloud-init`
3. [Follow the docs](https://www.latitude.sh/docs/servers/logging-into-your-server) on logging into your server
	- The username will be `ubuntu`
	- You will need the private key to match one of the public keys setup in the beginning of this doc or a key that matches an SSH keypair provided to the server by Latitude
	- If you enabled "all" services to be open from your IP, you can reach:
		- Grafana at `http://$YOUR_INSTANCE_IP:3000`
			- The username password will initially be the default `admin/admin`
			- You will be asked to reset these immediately after signin
		- Prometheus at `http://$YOUR_INSTANCE_IP:9090`