## Create external network

Inside `neutron_server` comtainer:

```bash
cat /etc/neutron/plugins/ml2/ml2_conf.in
```

Find:

```
[ml2_type_flat]
flat_networks = physnet1
```

So **provider-physical-network** is **physnet1**.

Create external network:

```bash
openstack network create --share --external --provider-network-type flat --provider-physical-network physnet1 external-net
```

## Create subnet

```bash
openstack subnet create --network external-net --subnet-range 192.168.100.0/24 --no-dhcp --gateway 192.168.100.1 external-subnet
```

## Create internal network

```bash
openstack network create network
```

## Create internal subnet

```bash
openstack subnet create subnet --network network --subnet-range 192.0.2.0/24
```

## CReate router

```bash
openstack router create router
```

Connect internal subnet:

```bash
openstack router add subnet router subnet
```

Conenect external network:

```bash
openstack router set router --external-gateway external-net
```

