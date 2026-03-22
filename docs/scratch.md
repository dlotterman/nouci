```
virsh destroy nouci1
virsh undefine nouci1
cp -f "$NUOCI_DATA_DIR"/images/noble-server-cloudimg-amd64.img  "$NUOCI_DATA_DIR"/images/nuoci1.img
sed -i '/localhost/d' ~/.ssh/known_hosts

~/.local/bin/yq4 eval-all '. as $item ireduce ({}; . *+ $item )' modules/init/nouci_init_root.yaml modules/base/nouci_base.yaml modules/disk_local/nouci_disk_local_single.yaml  modules/disk_encryption/nouci_disk_local_single_enc.yaml modules/node_exporter/nouci_node_exporter.yaml modules/prometheus/nouci_prometheus.yaml modules/grafana/nouci_grafana.yaml | tee nouci.yaml

virt-install --name nouci1 \
    --memory 4096 \
    --os-variant ubuntunoble \
    --disk=size=12,backing_store=""$NUOCI_DATA_DIR"/images/nuoci1.img" \
    --network default,mac=a2:34:89:d0:4d:69 \
    --cloud-init user-data="$(pwd)/nouci.yaml" \
    --noautoconsole

virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::2225-:22'
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::3001-:3000'
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::9100-:9100'
virsh qemu-monitor-command --hmp nouci1 'hostfwd_add ::9090-:9090'
ssh -p 2225 -i ~/.ssh/dlotterman_org ubuntu@localhost

```

virsh destroy nouci1
virsh undefine nouci1
cp -f "$NUOCI_DATA_DIR"/images/noble-server-cloudimg-amd64.img  "$NUOCI_DATA_DIR"/images/nuoci1.img


~/.local/bin/yq4 eval-all '. as $item ireduce ({}; . *+ $item )' modules/init/nouci_init_root.yaml modules/base/nouci_base.yaml modules/disk_local/nouci_disk_local_single.yaml  modules/disk_encryption/nouci_disk_local_single_enc.yaml modules/node_exporter/nouci_node_exporter.yaml modules/prometheus/nouci_prometheus.yaml modules/grafana/nouci_grafana.yaml | tee nouci.yaml

virt-install --name nouci1 \
    --memory 4096 \
    --os-variant ubuntunoble \
    --disk=size=12,backing_store=""$NUOCI_DATA_DIR"/images/nuoci1.img" \
    --network default,mac=a2:34:89:d0:4e:69 \
    --cloud-init user-data="$(pwd)/nouci.yaml" \
    --noautoconsole

virsh domifaddr --domain nouci1
sed -i '/94/d' ~/.ssh/known_hosts
ssh -i ~/.ssh/dlotterman_org ubuntu@192.168.122.167


```
virt-install --name nouci1 \
    --transient \
    --memory 4096 \
    --os-variant ubuntunoble \
    --disk=size=12,backing_store=""$NUOCI_DATA_DIR"/images/nuoci1.img" \
    --network default \
    --cloud-init user-data="$(pwd)/nouci.yaml","meta-data=$(pwd)/glue/meta-data","network-config=$(pwd)/glue/network-config" \
    --noautoconsole

virt-install --name nouci1 \
    --memory 4096 \
    --os-variant ubuntunoble \
    --disk=size=12,backing_store=""$NUOCI_DATA_DIR"/images/nuoci1.img" \
    --network default \
    --cloud-init user-data="$(pwd)/nouci.yaml","meta-data=$(pwd)/glue/meta-data","network-config=$(pwd)/glue/network-config" \
    --noautoconsole
```
