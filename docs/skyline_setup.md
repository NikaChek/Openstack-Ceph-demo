## Create the Skyline user

```bash
openstack user create --domain default --password-prompt skyline
```

## Add the admin role to the Skyline user

```bash
openstack role add --project service --user skyline admin
```

## Install MySQL/MariaDB

```bash
apt-get install mariadb-server -y
```

## Create database

```bash
sudo mysql
```

```sql
CREATE DATABASE IF NOT EXISTS skyline DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
```

## Grant access (replace `<DB_PASSWORD>`)

```sql
GRANT ALL PRIVILEGES ON skyline.* TO 'skyline'@'localhost' IDENTIFIED BY '<DB_PASSWORD>';
GRANT ALL PRIVILEGES ON skyline.* TO 'skyline'@'%'  IDENTIFIED BY '<DB_PASSWORD>';
FLUSH PRIVILEGES;
```

## Setup Skyline app

Pull Skyline service image from Docker Hub:

```bash
sudo docker pull 99cloud/skyline:latest
```

Clone skyline repository:

```bash
git clone https://opendev.org/openstack/skyline-apiserver
```

Create required folders:

```bash
sudo mkdir -p /etc/skyline /var/log/skyline /var/lib/skyline /var/log/nginx
```

Configure Skyline using `skyline.yaml` file:

```bash
cp skyline-apiserver/etc/skyline.yaml.sample /etc/skyline/skyline.yaml
```

Example changes (replace `<DB_PASSWORD>`, `<KEYSTONE_URL>`, `<SWIFT_URL>`):

```yaml
default:
  database_url: mysql+pymysql://skyline:<DB_PASSWORD>@localhost:3306/skyline
  
openstack:
  system_user_password: <SYSTEM_USER_PASSWORD>
  keystone_url: <KEYSTONE_URL>
externalSwift:
  enabled: true
  url: <SWIFT_URL>
```

## Bootstrap container

```bash
rm -rf /tmp/skyline && mkdir /tmp/skyline && mkdir /var/log/skyline
```

```bash
docker run -d --name skyline_bootstrap -e KOLLA_BOOTSTRAP="" \
  -v /etc/skyline/skyline.yaml:/etc/skyline/skyline.yaml \
  --net=host 99cloud/skyline:latest
```

Check logs:

```bash
docker logs skyline_bootstrap
```

If database migration succeeds, remove bootstrap container:

```bash
docker rm -f skyline_bootstrap
```

## Run Skyline

```bash
docker run -d --name skyline --restart=always \
  -v /etc/skyline/skyline.yaml:/etc/skyline/skyline.yaml \
  --net=host 99cloud/skyline:latest
```

## Port forwarding (example)

```bash
ssh -i ~/.ssh/<PRIVATE_KEY>.pem -L 9999:<CONTROLLER_IP>:9999 user@<BASTION_IP> -p <PORT>
```

Then Skyline is available at:
`http://localhost:9999`
