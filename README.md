# OpenStack + Ceph - Demo

This repo is a minimal OpenStack cluster deployed with kolla-ansible,
integrated with Ceph, and includes a basic admin demo:
Horizon access, networks/routers, CirrOS instance, Cinder volumes, and a quick
role/quotas walkthrough. Optional: Skyline dashboard and Swift (via RGW).

## Topology: 
1 controller,2 computes, Ceph cluster with 3 OSDs.

---

## Repo layout

```
.
├─ docs/
│  ├─ cephadm.md                 # Bootstrap Ceph with cephadm
│  ├─ delete_create_osd.md       # Replace OSDs (remove/add) via ceph orch
│  ├─ openstack_deployment.md    # kolla-ansible install, globals.yml, deploy
│  ├─ openstack_net.md           # External + internal networks, router, subnets
│  ├─ openstack_administration.md# Users, projects, roles, quotas, snapshots
│  ├─ swift.md                   # Enable Swift via Ceph RGW, endpoints, Horizon fix
│  └─ skyline_setup.md           # Skyline dashboard via Docker, DB bootstrap
├─ .gitignore
├─ LICENSE (MIT)
└─ README.md 
```

---

## Quickstart

1) **Ceph (cephadm)** — bootstrap, add hosts/OSDs, create pools, create OpenStack keyrings.  
   See: `docs/cephadm.md`

2) **kolla-ansible** — create venv, install kolla, copy configs, generate passwords, set `globals.yml`  
   with Ceph backends enabled and OVN, then:
   ```bash
   kolla-ansible bootstrap-servers -i multinode
   kolla-ansible prechecks -i multinode
   kolla-ansible deploy -i multinode
   ```
   See: `docs/openstack_deployment.md`

3) **Networking (OVN)** — create external flat network (`physnet1`), internal network, subnets, router.  
   See: `docs/openstack_net.md`

4) **Basic admin demo** — create project/user/role, assign role, set quotas, make instance snapshot.  
   See: `docs/openstack_administration.md`

5) **Verify Ceph integration** — confirm images/volumes/VM disks exist in RBD pools:
   ```bash
   rbd -p images ls
   rbd -p volumes ls
   rbd -p vms ls
   ```

6) **Swift via RGW** — enable rgw, add object-store endpoints, Keystone auth config, Horizon fix.  
   See: `docs/swift.md`

7) **Skyline** — standalone container + DB, point it to Keystone and (optionally) external Swift.  
   See: `docs/skyline_setup.md`
   
