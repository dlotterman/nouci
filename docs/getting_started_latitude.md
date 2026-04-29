# Getting started with Latitude.sh

[Latitude.sh](https://www.latitude.sh/) is a globally available Bare Metal Cloud platform which supports [cloud-init](https://cloud-init.io/).

Launching nouci with LSH should be straightforward:

* Substitute `open` for your editor of choice

```
git clone https://github.com/dlotterman/nouci && cd nouci
open init/nouci_init.yaml
```

Edit `nouci_init.yaml`:
- Provide your Github or Launchpad username
`ssh_import_id: ["gh:dlotterman"]`
	- This imports SSH keys based on [cloud-init](https://docs.cloud-init.io/en/latest/reference/yaml_examples/ssh.html#import-ssh-id)'s use of [ssh-import-id](https://manpages.ubuntu.com/manpages/jammy/man1/ssh-import-id.1.html)
- Punch a firewall hole for your IP:
`    - [ufw, allow, from, 35.149.59.78, to, any, port, 22]`

