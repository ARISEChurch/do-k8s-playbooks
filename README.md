# do-k8s-playbooks

Manage a Kubernetes cluster on Digitalocean using Ansible.


## Prepare

1. `npm install`
2. `export DO_API_TOKEN=xxx`
3. `ansible-galaxy install defunctzombie.coreos-bootstrap`

## Configuration

Copy `group_vars/all.sample` to `group_vars/all` and edit.

## Create a simple 3 node cluster

```sh
ansible-playbook -i inventories/digitalocean.sh create-master.yaml; ./add-node.sh; ./add-node.sh; ./add-node.sh
export FLEETCTL_TUNNEL=<master-ip>
fleetctl start services/master/*.service
fleetctl start services/node/*.service
ansible-playbook -i inventories/digitalocean.sh reboot.yaml
```

## Create a 3 node cluster with dns and ceph

```sh
ansible-playbook -i inventories/digitalocean.sh create-master.yaml; ./add-node.sh; ./add-node.sh; ./add-node.sh
export FLEETCTL_TUNNEL=<master-ip>
fleetctl start services/master/*.service
./setup-dns.sh
fleetctl start services/node/*.service
ansible-playbook -i inventories/digitalocean.sh bootstrap-ceph.yaml
fleetctl start services/ceph/ceph-mon.service
# Create 3 OSD's using ceph/base image on one of the nodes
# ssh core@<node ip> docker run --rm -v /etc/ceph:/etc/ceph --net=host ceph/base ceph osd create
# ssh core@<node ip> docker run --rm -v /etc/ceph:/etc/ceph --net=host ceph/base ceph osd create
# ssh core@<node ip> docker run --rm -v /etc/ceph:/etc/ceph --net=host ceph/base ceph osd create
fleetctl start services/ceph/ceph-osd.service
fleetctl start services/ceph/ceph-mds.service
# Once mds has bootstraped cephfs
fleetctl start services/ceph/mnt-cephfs.mount
```


## Other tips

### Load balance http pods

```sh
ssh core@<master-ip>
etcdctl set /vulcand/backends/default-<pod name>/backend '{"Type": "http"}'
etcdctl set /vulcand/frontends/default-<pod name>/frontend '{"Type": "http", "BackendId": "default-<pod name>", "Route": "Host(`host.domain.com`)"}'
```

### Forward the master api port

```sh
ssh -nNTL 8080:127.0.0.1:8080 core@<master-ip>
```

### Setup cluster DNS

First forward the master api port as above.

```sh
./setup-dns.sh
ansible-playbook -i inventories/digitalocean.sh reboot.yaml
```

### Setup ceph cluster

First create a 3 node cluster

```sh
ansible-playbook -i inventories/digitalocean.sh bootstrap-ceph.yaml
fleetctl start services/ceph/ceph-mon.service
# Create 3 OSD's using ceph/base image on one of the nodes
fleetctl start services/ceph/ceph-osd.service
fleetctl start services/ceph/ceph-mds.service
# After setting up cephfs
fleetctl start services/ceph/mnt-cephfs.mount
```

### SSL keys and certs

HTTP proxy layer is handled by `vulcand`. You can view there documentation
around SSL certs here: http://vulcand.io/proxy.html#managing-certificates
