---
title: Deploy Data Migration Using binary
summary: Use binary to deploy the Data Migration cluster.
category: tools
---

# Deploy Data Migration Using binary

## Download binary

```shell
# Download the DM package.
wget http://download.pingcap.org/dm-latest-linux-amd64.tar.gz
wget http://download.pingcap.org/dm-latest-linux-amd64.sha256

# Check the file integrity. If the result is OK, the file is correct.
sha256sum -c dm-latest-linux-amd64.sha256
# Extract the package.
tar -xzf dm-latest-linux-amd64.tar.gz
cd dm-latest-linux-amd64
```

## Config File

* all sample config files can be found in directory `conf` of dm tarball
* sample config file of dm-master: `bin/dm-master -print-sample-config`
* sample config file of dm-worker: `bin/dm-worker -print-sample-config`

## Deployment


### the DM cluster topology

| Name | Host IP | Services |
| ---- | ------- | -------- |
| node1 | 172.16.10.71 | DM-master|
| node2 | 172.16.10.72 | DM-worker1-1, DM-worker1-2 |
| node3 | 172.16.10.73 | DM-worker2-1, DM-worker2-2 |

### Start all DM-worker

```shell
bin/dm-master \
    --master-addr="172.16.10.71:8261"\
    -L="info" \
    --config="{{ dm_master_config_file }}" \
    --log-file="{{ dm_master_log_dir }}/{{ dm_master_log_file }}" 2>> "{{ dm_master_log_dir }}/{{ dm_master_stderr_log_file }}"

```

config file of DM-master

```toml
# Master Configuration.

# log configuration
#log-level = "info"
#log-file = "dm-master.log"

# dm-master listen address
#master-addr = "172.16.10.71:8261"

# replication group <-> dm-Worker deployment, we'll refine it when new deployment function is available
[[deploy]]
source-id = "mysql-replica-11"
dm-worker = "172.16.10.72:8262"

[[deploy]]
source-id = "mysql-replica-12"
dm-worker = "172.16.10.72:8263"

[[deploy]]
source-id = "mysql-replica-21"
dm-worker = "172.16.10.73:8262"

[[deploy]]
source-id = "mysql-replica-22"
dm-worker = "172.16.10.73:8263"
```

### Start DM-master

Take `DM-worker1-1` as an example

```shell
bin/dm-worker \
    --worker-addr="172.16.10.72:8262" \
    -L="info" \
    --relay-dir="{{ dm_worker_relay_dir }}" \
    --config="{{ dm_worker_config_file }}" \
    --log-file="{{ dm_worker_log_dir }}/{{ dm_worker_log_file }}"  2>> "{{ dm_worker_log_dir }}/{{ dm_worker_stderr_log_file }}"
```

config file of `DM-worker1-1`

```toml
# Worker Configuration.

# log configuration
#log-level = "info"
#log-file = "dm-worker.log"

# dm-worker listen address
#worker-addr = ":8262"

# server id of slave for binlog replication
# each instance (master and slave) in replication group should have different server id
server-id = 101

# represents a MySQL/MariaDB instance or a replication group
source-id = "mysql-replica-11"

# flavor: mysql/mariadb
flavor = "mysql"

# directory that used to store relay log
#relay-dir = "./relay_log"

#  enable gtid in relay log unit
enable-gtid = false

# charset of DSN of source mysql/mariadb instance
# charset= ""

[from]
host = "127.0.0.1"
user = "root"
password = ""
port = 3306

# relay log purge strategy
#[purge]
#interval = 3600
#expires = 24
```