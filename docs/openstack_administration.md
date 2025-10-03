## Create project, user and role

Create project: 

```bash
openstack project create --description 'my new project' new-project
```

Create user:

```bash
openstack user create --project new-project --password PASSWORD new-user
```

Create role:

```bash
openstack role create new-role
```

Assign a role to a user-project pair:

```bash
openstack role add --user USER_NAME --project TENANT_ID ROLE_NAME
```

See the role assignment:

```bash
openstack role assignment list --user USER_NAME --project PROJECT_ID --names
```

## Role assignment

I create project `new-project` with user `new-user` and role `my-role`. To change permission of this role, I modify `policy.yaml` files in corresponding Docker container.

To generate `policy.yaml` file:

```bash
oslopolicy-policy-generator --namespace <service_name>
```

For example:

```bash
oslopolicy-policy-generator --namespace keystone
```

List of services:

```bash
openstack service list
```

My output (with small description):

```bash
+----------------------------------+-----------+----------------+
| ID                               | Name      | Type           |
+----------------------------------+-----------+----------------+
| 1ee65459c04c4b98b76570f8e0d834bb | cinderv3  | volumev3       | # Provides block storage volumes for instances
| 2cd02973480a42b38925713619348165 | heat-cfn  | cloudformation | # AWS CloudFormation-compatible API for orchestration
| 7516e2e2cc9e46e694235a4c97cd5d88 | neutron   | network        | # Provides networking-as-a-service
| 937841b874314ffda635069a84d4c5fa | nova      | compute        | # Manages virtual machines
| 93f3c59ac4b6434192ac1297ab63917c | glance    | image          | # Stores and manages disk images
| 997ff4e52eb14778acb080834b5c3a13 | heat      | orchestration  | # Orchestrates complex infrastructure using templates
| b4716292ef3d4541908c402e04b45b7a | keystone  | identity       | # Authentication, authorization, and user/project/role management
| cdeb2140f4554e519b44277b3e2a1b53 | placement | placement      | # Tracks resource inventory and usage for efficient VM scheduling
| d885691de4a548ecac2fdd6034f2fcec | swift     | object-store   | Object storage for large-scale unstructured data
+----------------------------------+-----------+----------------+
```

Then add this policy file in conf `/etc/<service_name>/<service_name>.conf`:

```bash
[oslo_policy]
policy_file = /etc/<service_name>/policy.yaml
```

Restart container.

## My role changes

### Keystone

identity:list_roles (role list)
identity:list_users (user list)

### Nova

os_compute_api:servers:detail (server list)

### Neutron

All can list networks/subnets/routers/ports, but see not all networks).

## Quotas

List all quotas:

```bash
openstack quota show
```

Example of hanging quota:

```bash
openstack quota set --instances 5
```

## Snapshot of instance

```bash
openstack server image create --name <snapshot-name> <server-name>
```

