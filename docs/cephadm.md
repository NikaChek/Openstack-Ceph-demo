## Main node

Install `cephadm`

```sh
apt install -y cephadm
cephadm install ceph-common
```

Bootstrap

```sh
cephadm bootstrap --mon-ip *<mon-ip>*
```

Enable Ceph CLI

```sh
cephadm shell
```

## Adding hosts

Copy `/etc/ceph/ceph.pub` from main node to `/root/.ssh/authorized_keys` for other hosts

Add host

```sh
ceph orch host add *<newhost>* [*<ip>*] [*<label1> ...*]
```

## Add osd

```sh
ceph orch daemon add osd host:path-to-disk
```

## Create fs

Create fs

```sh
ceph osd pool create cephfs_data
ceph osd pool create cephfs_metadata
```

```sh
ceph fs new <fs_name> <metadata> <data>
```

Create client

```sh
ceph auth get-or-create client.<name>
```

Change permissions

```sh
ceph auth caps client.<name> DAEMON_TYPE 'allow CAPABILITY' [DAEMON_TYPE 'allow CAPABILITY']
```

For example, for rw to fs `cephfs`:

```sh
ceph auth caps client.2 mds 'allow rw' mon 'allow r' osd 'allow rw pool=cephfs_data'
```

Copy client auth info:

```sh
ceph auth get client.2
```

to `/etc/ceph/ceph.client.<name>.keyring` on client machine.

Mount dir to fs:

```sh
sudo ceph-fuse -n client.<name> --client_fs <cephfs_name> /mnt/path
```

If client permission was changed, then 

```sh
sudo umount /mnt/path
```

and remount.

## Create pools and clients for OpenStack

Create pools:

```bash
ceph osd pool create volumes
ceph osd pool create images
ceph osd pool create backups
ceph osd pool create vms

rbd pool init volumes
rbd pool init images
rbd pool init backups
rbd pool init vms
```

Create keyrings:

```bash
ceph auth get-or-create client.glance mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=images' -o /etc/ceph/ceph.client.glance.keyring
ceph auth get-or-create client.cinder mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=images' -o /etc/ceph/ceph.client.cinder.keyring
ceph auth get-or-create client.nova mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=vms, allow rx pool=images' -o /etc/ceph/ceph.client.nova.keyring
ceph auth get-or-create client.cinder-backup mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=backups' -o /etc/ceph/ceph.client.cinder-backup.keyring
```
