# Kine With YugabyteDB
==========================

This fork adds support for YugabyteDB backend. The YugabyteDB version is based on the PostgreSQL backend with several optimizations relevant for distributed SQL databases.

To use the YugabyteDB backend, pass the `yugabytedb` as a driver name in your connection string. For instance, a complete connection string can look as follows:
```sql
yugabytedb://yugabyte:yugabyte@127.0.0.1:5433/yugabyte
```

A YugabyteDB cluster needs to be started with the [ysql_sequence_cache_minval=1](https://docs.yugabyte.com/preview/reference/configuration/yb-tserver/#ysql-sequence-cache-minval) TServer flag. By default, YugabyteDB creates sequences with `CACHE=100` which means that one connection to the database can insert records with ids in the range from 1 to 100 while the other in the range from 101 to 200. This can lead to `version mismatch` errors on the Kubernetes end that depends on sequences that increment strictly by `1` (meaning the sequences need to be created with `CACHE=1`). Here is an example for Docker that start a YugabyteDB node using the `yugabyted` tool:
```shell
docker run -d --name yugabytedb_node1 --net custom-network \
  -p 15433:15433 -p 7001:7000 -p 9000:9000 -p 5433:5433 \
  -v ~/yb_docker_data/node1:/home/yugabyte/yb_data --restart unless-stopped \
  yugabytedb/yugabyte:latest \
  bin/yugabyted start --tserver_flags="ysql_sequence_cache_minval=1" \
  --base_dir=/home/yugabyte/yb_data --daemon=false
```

---

Kine is an etcdshim that translates etcd API to:
- SQLite
- Postgres
- MySQL
- YugabyteDB
- NATS Jetstream

## Features
- Can be ran standalone so any k8s (not just K3s) can use Kine
- Implements a subset of etcdAPI (not usable at all for general purpose etcd)
- Translates etcdTX calls into the desired API (Create, Update, Delete)

See an [example](/examples/minimal.md).
