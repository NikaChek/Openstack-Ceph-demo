## Insatall dependencies

```bash
sudo apt install git python3-dev libffi-dev gcc libssl-dev libdbus-glib-1-dev
```

## Virtual environment

```bash
sudo apt install python3-venv
python3 -m venv /path/to/venv
source /path/to/venv/bin/activate
pip install -U pip
```

## Install `kolla-ansible`

```bash
pip install git+https://opendev.org/openstack/kolla-ansible@master
```

```bash
sudo mkdir -p /etc/kolla
```

Copy files

```bash
cp -r /path/to/venv/share/kolla-ansible/etc_examples/kolla/* /etc/kolla
```

```bash
cp /path/to/venv/share/kolla-ansible/ansible/inventory/multinode .
```

## Install Ansible Galaxy

```bash
kolla-ansible install-deps
```

## Generate passwords

```bash
kolla-genpwd
```

## Kolla `multinode`

Edit `./multinode` file.

Check connection:

```bash
ansible -i multinode all -m ping
```

## Create directories and files 

```bash
mkdir /etc/kolla/config
mkdir /etc/kolla/config/nova
mkdir /etc/kolla/config/glance
mkdir -p /etc/kolla/config/cinder/cinder-volume
mkdir /etc/kolla/config/cinder/cinder-backup
```

Copy `ceph.conf` and keyrings:

```bash
cp /etc/ceph/ceph.conf /etc/kolla/config/cinder/
cp /etc/ceph/ceph.conf /etc/kolla/config/nova/
cp /etc/ceph/ceph.conf /etc/kolla/config/glance/
cp /etc/ceph/ceph.client.glance.keyring /etc/kolla/config/glance/
cp /etc/ceph/ceph.client.nova.keyring /etc/kolla/config/nova/
cp /etc/ceph/ceph.client.cinder.keyring /etc/kolla/config/nova/
cp /etc/ceph/ceph.client.cinder.keyring /etc/kolla/config/cinder/cinder-volume/
cp /etc/ceph/ceph.client.cinder.keyring /etc/kolla/config/cinder/cinder-backup/
cp /etc/ceph/ceph.client.cinder-backup.keyring /etc/kolla/config/cinder/cinder-backup/
```

And copy /etc/ceph to other controller and compute nodes.

## Kolla `globals.yaml`

Location: `/etc/kolla/globals.yaml`

```bash
workaround_ansible_issue_8743: yes
config_strategy: "COPY_ALWAYS"
kolla_base_distro: "ubuntu"
openstack_release: "2024.1"
node_custom_config: "/etc/kolla/config"
kolla_internal_vip_address: "192.168.100.101"
kolla_container_engine: docker
network_interface: "ens3"
neutron_external_interface: "ens4"
enable_openstack_core: "yes"
enable_haproxy: "no"
enable_ceph_rgw: "yes"
enable_ceph_rgw_loadbalancer: "no"
enable_ceph_rgw_keystone: "yes"
enable_cinder: "yes"
cinder_cluster_skip_precheck: "true"
enable_etcd: "yes"
enable_fluentd: "no"
ceph_glance_user: "glance"
ceph_glance_keyring: "client.glance.keyring"
ceph_glance_pool_name: "images"
ceph_cinder_user: "cinder"
ceph_cinder_pool_name: "volumes"
ceph_cinder_backup_user: "cinder-backup"
ceph_cinder_backup_pool_name: "backups"
ceph_cinder_keyring: "client.cinder.keyring"
ceph_cinder_backup_keyring: "client.cinder-backup.keyring"
ceph_nova_user: "{{ ceph_cinder_user }}"
ceph_nova_pool_name: "vms"
glance_backend_ceph: "yes"
cinder_backend_ceph: "yes"
cinder_coordination_backend: "etcd"
nova_backend_ceph: "yes"
nova_console: "novnc"
```

## Deployment

```bash
kolla-ansible bootstrap-servers -i ./multinode
kolla-ansible prechecks -i ./multinode
kolla-ansible deploy -i ./multinode
```


