# Getting started with libvirt

libvirt, also sometimes referred as "KVM" or "Linux Virtualization", is a ecosystem of coordinated tools focused on provide production ready virtualization stacks. libvirt is a library for much of the rest of the Linux virtualization ecosystem.

This guide should work with any modern Linux distrobution, and should work with "[rootless](https://developers.redhat.com/articles/2024/12/18/rootless-virtual-machines-kvm-and-qemu#libvirt_system_and_session_interfaces)" or "[user](https://bitsanddragons.wordpress.com/2025/01/14/rootless-virtual-machines-with-kvm-and-qemu-on-opensuse-15-5/)" libvirt.


# Edit and Build:

**Clone** this nouci repository and change directory to desired Ubuntu release, 2404 assumed:

```
git clone https://github.com/dlotterman/nouci && cd nouci/ubuntu_releases/2404
```

open init/nouci_init.yaml (Substitute `open` for your editor of choice)
```
open init/nouci_init.yaml
```

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

### Download and prepare LTS cloud image

```
NOUCI_DATA_DIR=/var/tmp/nouci
mkdir -p "$NOUCI_DATA_DIR"/images/
wget -O "$NOUCI_DATA_DIR"/images/noble-server-cloudimg-amd64.img  https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
cp -f "$NOUCI_DATA_DIR"/images/noble-server-cloudimg-amd64.img  "$NOUCI_DATA_DIR"/images/nouci1.img
```

* `/var/tmp` is chosen because its user writeable but outside users home directory, replace as needed.

### Optional(ish): Install yq4
```
mkdir -p ~/.local/bin
wget -O ~/.local/bin/yq4 https://github.com/mikefarah/yq/releases/download/v4.52.4/yq_linux_amd64
chmod +x ~/.local/bin/yq4
```

## Configure nouci
Edit the init file.

For insecure / debug installations:

```
open modules/init/nouci_init_root.yaml
```


## Build nouci.yaml
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

## Launch nouci1
```
virt-install --name nouci1 \
    --memory 4096 \
    --os-variant ubuntunoble \
    --disk=size=12,backing_store=""$NOUCI_DATA_DIR"/images/nouci1.img" \
    --network default \
    --cloud-init user-data="$(pwd)/nouci.yaml" \
		--autostart \
    --noautoconsole
```

### Updates

The cloud-init will instruct the base OS to upgrade its packages, and if needed, reboot.

Since we didn't apply `--autostart` above, the VM needs to be started again from its reboot.
* This is a todo, the right flags for the right sequence here should be specified to have the VM stay alive as much as possible

```
virsh start nouci1
```

If rootfull (may need to wait ~10seconds for VM to boot):

```
virsh domifaddr --domain nouci1
sed -i '/192.168.122.94/d' ~/.ssh/known_hosts
ssh -i ~/.ssh/dlotterman_org ubuntu@192.168.122.94
```

If rootless:

```
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::2222-:22'
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::3000-:3000'
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::9100-:9100'
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::9090-:9090'
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::9115-:9115'
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::9116-:9116'
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::9089-:9089'
sed -i '/localhost/d' ~/.ssh/known_hosts
ssh -i ~/.ssh/dlotterman_org -p 2222 ubuntu@localhost
```

rootless network info:

```
$ virsh qemu-monitor-command --domain nouci1 --hmp info network
net0: index=0,type=nic,model=virtio-net-pci,macaddr=a2:34:89:d0:4d:69
 \ hostnet0: index=0,type=user,net=10.0.2.0,restrict=off

 $ virsh qemu-monitor-command --domain nouci1 --hmp info usernet
 Hub -1 (hostnet0):
   Protocol[State]    FD  Source Address  Port   Dest. Address  Port RecvQ SendQ
   TCP[HOST_FORWARD] 101               *  2225       10.0.2.15    22     0     0
 ```

You are done! Access grafana on port `:3000`!
* Username: `admin`
* Password: `admin` (it will ask you to reset	)

To close your lab:

```
virsh destroy nouci1
virsh undefine nouci1 --remove-all-storage
```

Optionally, remove the ubuntu-cloud image:

```
NOUCI_DATA_DIR=/var/tmp/nouci
rm -rf $NOUCI_DATA_DIR
```