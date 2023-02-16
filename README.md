# ansible

This is a set of playbooks we use to bootstrap everything.

## Things to consider

This Ansible set of playbooks assumes you're using `zsh` as your shell and adds all aliases to `~/.zshrc`, so consider updating it if you use another shell.

## Prerequisites:

0. Obviously you need Ansible itself installed.

1. You need to install a node_exporter role by running `ansible-galaxy install cloudalchemy.node_exporter` and install [its requirements](https://github.com/cloudalchemy/ansible-node-exporter#requirements) in case you would want to install node_exporter there.

2. You need to create `hosts.txt` with all of your hosts, here's an example:

```
[validators]
validator-1      ansible_host=192.168.1.1

[testnets]
validator-2      ansible_host=192.168.1.2

[fullnodes:children]
validators
testnets

[chain_1]
validator-1
validator-2

[fullnodes:vars]
ansible_user=<your hostname>
ansible_ssh_private_key_file=<your SSH keyfile>
```

Specifically, you need to group all your hosts by chain (put all of the hosts on one chain under a group), and all hosts should be under `fullnodes` groups (either directly or indirectly). We personally use `validators` and `testnets` groups for mainnet and testnet validators, and `monitoring` for fullnodes used not for validators.

3. You need to specify the variables for each chain which will be later used for bootstrapping a node, such as github repository path, binary version, where to put a binary, minimal gas prices, seeds, peers, genesis path etc. Check out the files in `group_vars/` for reference.

You should put the variables into the following files:
- `group_vars/all` for variables needed for all servers
- `group_vars/fullnodes` for variables needed for fullnodes
- `group_vars/<chain-name>` for variables for fullnodes for specific chain
- `host_vars/<host-name>` for variables for specific host (like `ansible_become_pass`)


4. You now may run playbooks to bootstrap it. Each playbook accepts `-e "hosts=<hosts to run at>"` argument, otherwise it'll run on all hosts.

## Bootstrapping a server

1. Install [node_exporter](https://github.com/prometheus/node_exporter), enables and starts it:

```
ansible-playbook playbooks/server/01-node-exporter.yml --extra-vars "hosts=validator-1"
```

2. Install zsh, antigen, powerlevel10k and sets it all up:

```
ansible-playbook playbooks/server/02-zsh.yml --extra-vars "hosts=validator-1"
```


## Bootstrapping a full node

This is a set of playbooks to bootstrap a single fullnode on any cosmos-sdk based chain.

1. Install Golang binary (needed to compile a fullnode binary from source):

```
ansible-playbook playbooks/fullnode/01-golang.yml --extra-vars "hosts=validator-1"
```

2. Fetch and install latest fullnode binary:

```
ansible-playbook playbooks/fullnode/02-install-fullnode-binary.yml --extra-vars "hosts=validator-1"
```

3. Init a fullnode and set up its config.

```
ansible-playbook playbooks/fullnode/03-setup-fullnode.yml --extra-vars "hosts=validator-1"
```

4. Download Cosmovisor, set up its folder layout and setup a systemd service for it:

```
ansible-playbook playbooks/fullnode/04-install-cosmovisor.yml --extra-vars "hosts=validator-1"
```

5. Set up statesync:

```
ansible-playbook playbooks/fullnode/05-statesync.yml --extra-vars "hosts=validator-1"
```

6. Download, build and install [tendermint-exporter](https://github.com/solarlabsteam/tendermint-exporter):

```
ansible-playbook playbooks/fullnode/06-tendermint-exporter.yml --extra-vars "hosts=validator-1"
```

7. Restarts the full node or starts it if it was stopped.

```
ansible-playbook playbooks/fullnode/07-start-node.yml --extra-vars "hosts=validator-1"
```

8. Get node status (useful to see the info from multiple servers)

```
ansible-playbook playbooks/fullnode/08-node-status.yml --extra-vars "hosts=fullnodes"
```

9. Prepare for a chain upgrade using Cosmovisor

Need to specify upgrade_name (can be taken from the proposal) and upgrade_version (can be taken from the instructions or Github). This will fetch the required binary, build it, create a folder layout for the upgrade, put the binary there and print its version.

```
ansible-playbook playbooks/fullnode/09-prepare-cosmovisor-upgrade.yml --extra-vars "hosts=validator-1 upgrade_name=v2 upgrade_version=v2.0.0"
```