## Enable RGW

```bash
ceph mgr module enable rgw
```

```bash
ceph orch apply rgw
```

## Change `globals.yaml`

Add line:

```bash
enable_swift: "yes"
```

Reconfigure:

```bash
kolla-ansible reconfigure -i multinode
```

## ERROR

Horizon container was unhealthy. I find error in `/var/log/kolla/horizon/horizon-error.log`

```bash
grep -r WSGIScriptAlias /etc/apache2
```

Here was `/var/lib/kolla/venv/lib/python3/site-packages/openstack_dashboard/wsgi.py` but should be `/var/lib/kolla/venv/lib/python3.12/site-packages/openstack_dashboard/wsgi.py`.

So I change `/etc/apache2/conf-enabled/000-default.conf` and run

```bash
apache2ctl restart
```

## Add endpoints

```bash
openstack endpoint create --region RegionOne object-store public "http://192.168.100.102:80/swift/v1/AUTH_%(project_id)s"
openstack endpoint create --region RegionOne object-store internal "http://192.168.100.102:80/swift/v1/AUTH_%(project_id)s"
openstack endpoint create --region RegionOne object-store admin "http://192.168.100.102:80/swift/v1/AUTH_%(project_id)s"
```

## Add keystone settings

Find services:

```bash
ceph orch ps --daemon_type=rgw
```

Add settings for each service:

```bash
ceph config set client.rgw.my-rgw.ceph1.rjymsf rgw_keystone_url https://192.168.100.101:5000
ceph config set client.rgw.my-rgw.ceph1.rjymsf rgw_keystone_api_version 3
ceph config set client.rgw.my-rgw.ceph1.rjymsf rgw_keystone_admin_user swift
ceph config set client.rgw.my-rgw.ceph1.rjymsf rgw_keystone_admin_password password-for-swift
ceph config set client.rgw.my-rgw.ceph1.rjymsf rgw_keystone_admin_domain default
ceph config set client.rgw.my-rgw.ceph1.rjymsf rgw_keystone_admin_project service
ceph config set client.rgw.my-rgw.ceph1.rjymsf rgw_s3_auth_use_keystone true
ceph config set client.rgw.my-rgw.ceph1.rjymsf rgw_keystone_implicit_tenants false
ceph config set client.rgw.my-rgw.ceph1.rjymsf rgw_keystone_accepted_roles admin,member,_member_
ceph config set client.rgw.my-rgw.ceph1.rjymsf rgw_keystone_verify_ssl false
ceph config set client.rgw.my-rgw.ceph1.rjymsf rgw_swift_account_in_url true
ceph config set client.rgw.my-rgw.ceph1.rjymsf rgw_enable_apis swift,s3
```

Redeploy:

```bash
ceph orch redeploy rgw.my-rgw
```

Check:

```bash
ceph config dump
```

## Use

```bash
openstack container list
```

or

```bash
swift list
swift list test
swift upload test test.txt
swift download test test.txt
```

## Skyline

Add in `/etc/skyline/skyline.yaml`:

```bash
externalSwift:
  enabled: true
  url: http://192.168.100.102:80/swift/v1
```

And recreate container as described in `skyline_setup.md`.
