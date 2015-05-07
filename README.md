# do-k8s-playbooks

Manage a k8s cluster on Digitalocean using Ansible.

Comes with https://github.com/ARISEChurch/kubernetes-reverseproxy running on all
nodes to load balance your services with `nginx`. Read the README.md for usage
detail. It works best with round-robin DNS pointing to all your Kubernetes
nodes!

## Prepare

1. `npm install`
2. `export DO_API_TOKEN=xxx`
3. `ansible-galaxy install defunctzombie.coreos-bootstrap`

## Configuration

Copy `group_vars/all.sample` to `group_vars/all` and edit.

## Create a 3 node cluster

```sh
ansible-playbook -i inventories/digitalocean.sh create-master.yaml; ./add-node.sh; ./add-node.sh
fleetctl --tunnel <master-ip> start services/master/*.service
fleetctl --tunnel <master-ip> start services/node/*.service
```
