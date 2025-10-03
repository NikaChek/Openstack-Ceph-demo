## Delete OSD

```bash
ceph osd out {osd-num}
```

```bash
ceph -w
```

Wait.

```bash
ceph osd crush remove osd.{osd-num}
ceph auth del osd.{osd-num}
ceph osd rm {osd-num}
ceph orch daemon rm osd.{osd-num} --force
```

## Create OSD

```bash
ceph orch daemon cerate {hostname}:/path/to/disk
```

