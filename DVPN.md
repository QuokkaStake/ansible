# Bootstrapping your dVPN node using Ansible

This may come in handy if you have a lot of dVPN nodes or are planning to have a lot of them and want to provision or update them all at once.

### Prerequisites

You need Ansible, a bunch of servers to provision and their credentials, and a wallet with at least 100 dVPN per each dVPN node.

### Setting it up

1. Clone this repository
2. Create a `hosts.txt` file in the root of this repo, adding the future dVPN hosts there in the following way:

```
[dvpn_nodes]
sentinel-dvpn-1      ansible_host=<IP1>
sentinel-dvpn-2      ansible_host=<IP2>
...
```

This file is an Ansible inventory with 2 servers under `dvpn_nodes` group.

3. Create `group_vars/dvpn_nodes` file, which will hold all variables applied to `dvpn_nodes` group, it should have the content similar to that:

```
---
ansible_user: validator
ansible_ssh_private_key_file: ~/.ssh/id_rsa

```

Here, it's assumed that on every host, the SSH user is called `validator` and the key path is `~/.ssh/id_rsa`. If it's different per every host, you can override these variables in `host_vars/<hostname>` per each host.

3. Create `host_vars/<hostname>` per each host (so for this example, `host_vars/sentinel-dvpn-1` and `host_vars/sentinel-dvpn-2`), which will hold the variables for specific hosts (like SSH sudo password, moniker name, node price and wallet mnemonic). It should have the content similar to this:

```
---
ansible_become_pass: testtesttest
price: 1500000udvpn
moniker: dVPN node - 
mnemonic: never gonna give you up never gonna let you down never gonna run around and desert you never gonna make you cry never gonna say
key_name : badger
```

As you see, this holds SSH password (`ansible_become_pass`), node price, moniker and wallet mnemonic.

4. You are good to go!

### Bootstrapping the nodes

1. You need to install Docker on all hosts. This can be done via ansible-playbook this way:

```
ansible-playbook playbooks/dvpn/01-install-docker.yml -e "hosts=dvpn_nodes"
```

This will install Docker on all servers and reboot them.
If you want to provision a specific server, just pass it as a `hosts` variable this way:

```
ansible-playbook playbooks/dvpn/01-install-docker.yml -e "hosts=sentinel-dvpn-1"
```

2. Then, you need to run a playbook that will install the node

```
ansible-playbook playbooks/dvpn/02-install-and-setup-node.yml -e "hosts=dvpn_nodes"
```

This playbook will do the following:
- stop the current running node, if present
- fetch the latest binary
- build it (if there's no image built or it's not the latest)
- set up dvpn-node and Wireguard configs
- import wallet, if needed
- starts the dVPN node

### Updating the dVPN node

Updating can be done using the same playbook as the one used for installing a node from scratch:

```
ansible-playbook playbooks/dvpn/02-install-and-setup-node.yml -e "hosts=dvpn_nodes"
```
