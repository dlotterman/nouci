# Getting started with libvirt

```
git clone https://github.com/dlotterman/nouci && cd nuoci
open init/nouci_init.yaml
```
* Substitue `open` for your editor of choice

Edit `nouci_init.yaml` to provide either your [Github](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) (`gh:`) or [Launchpad](https://documentation.ubuntu.com/launchpad/user/how-to/import-ssh-keys/) (`lp:`) username from which to source SSH keys. If you feel inclined, you can also replace this with your own SSH `clout-init`.


```
NUOCI_DATA_DIR=/var/tmp/nuoci
mkdir -p "$NUOCI_DATA_DIR"/images/
wget -O "$NUOCI_DATA_DIR"/images/noble-server-cloudimg-amd64.img  https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
cp -f "$NUOCI_DATA_DIR"/images/noble-server-cloudimg-amd64.img  "$NUOCI_DATA_DIR"/images/nuoci1.img
```
* `/var/tmp` is chosen because its user writeable but outside users home directory, replace as needed.

### Optional(ish): Install yq4
```
wget -O ~/.local/bin/yq4 https://github.com/mikefarah/yq/releases/download/v4.52.4/yq_linux_amd64
chmod +x ~/.local/bin/yq4
```


## Build nouci.yaml
```
 ~/.local/bin/yq4 eval-all '. as $item ireduce ({}; . *+ $item )' modules/init/nouci_init_root.yaml modules/base/nouci_base.yaml modules/disk_local/nouci_disk_local_single.yaml  modules/disk_encryption/nouci_disk_local_single_enc.yaml modules/node_exporter/nouci_node_exporter.yaml modules/prometheus/nouci_prometheus.yaml modules/grafana/nouci_grafana.yaml | tee nouci.yaml
```

## Launch nouci1
```
virt-install --name nouci1 \
    --memory 4096 \
    --os-variant ubuntunoble \
    --disk=size=12,backing_store=""$NUOCI_DATA_DIR"/images/nuoci1.img" \
    --network default \
    --cloud-init user-data="$(pwd)/nouci.yaml" \
    --noautoconsole
```

If rootfull:
```
virsh domifaddr --domain nouci1
sed -i '/192.168.122.94/d' ~/.ssh/known_hosts
ssh -i ~/.ssh/dlotterman_org ubuntu@192.168.122.94
...

If rootless:
```
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::2225-:22'
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::3000-:3000'
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::9100-:9100'
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::9090-:9090'
sed -i '/localhost/d' ~/.ssh/known_hosts
ssh -i ~/.ssh/dlotterman_org -p 2225 ubuntu@localhost
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

To close your lab:
```
virsh destroy nouci1
virsh undefine nouci1 --remove-all-storage
```
