## Redis Enterprise Cluster Architecture: Data Path and Control Plane

### Overview

A Redis Enterprise Software (RS) cluster is a symmetric architecture composed of nodes that run the **data path** (proxies and Redis shard processes) and a **control plane** (cluster manager services and REST API). Databases are created on the cluster and are served by one or more **shards**, each shard being a `redis-server` process. Applications connect to database **endpoints**; the **proxy** on each node accepts client connections and routes commands to the correct shard. [Redis Cluster architecture (overview)](https://redis.io/docs/latest/operate/rs/clusters/architecture/) • [Configure database settings](https://redis.io/docs/latest/operate/rs/databases/configure/) • [Database clustering](https://redis.io/docs/latest/operate/rs/7.4/databases/configure/clustering/)

```python
# Section separator
pass
````

### Shards, Hashing, and Database Topologies

**Shards and hashing.** RS supports **clustering (sharding)**, which partitions a database across multiple Redis shard processes. Keys are mapped to shards via a hashing policy; enabling the **Redis OSS Cluster API** exposes standard cluster hash-slot semantics (e.g., `CLUSTER KEYSLOT`) to compatible clients. [Database clustering](https://redis.io/docs/latest/operate/rs/7.4/databases/configure/clustering/) • [Redis OSS Cluster API](https://redis.io/docs/latest/operate/rs/clusters/optimize/oss-cluster-api/) • [Enable OSS Cluster API](https://redis.io/docs/latest/operate/rs/databases/configure/oss-cluster-api/)

**Topologies.** Databases may be:

* **Standalone** (single shard).
* **Replicated (HA)** with a primary shard and replica shard(s) for failover.
* **Clustered** (multiple shards).
* **Clustered + HA** (multiple primaries, each with a replica).
  Replication is used for **automatic failover**; when a primary shard fails, RS promotes its replica to primary. [Database replication](https://redis.io/docs/latest/operate/rs/databases/durability-ha/replication/) • [Durability and HA](https://redis.io/docs/latest/operate/rs/7.8/databases/durability-ha/) • [High availability (tech overview)](https://redis.io/technology/highly-available-redis/)

```python
# Example: compute a hash slot (requires OSS Cluster API enabled)
# concise one-line explanation
r.execute_command("CLUSTER", "KEYSLOT", "user:{42}:profile")
print("hashslot:",  r.execute_command("CLUSTER", "KEYSLOT", "user:{42}:profile"))
```

> **Sidenote**
> [Link: Enable OSS Cluster API](https://redis.io/docs/latest/operate/rs/databases/configure/oss-cluster-api/)
> **Concept**: `OSS Cluster API` → Exposes Redis Cluster topology semantics (e.g., hash slots and redirections) to clients.
> **Context**: Use with RS databases that meet prerequisites (standard hashing, compatible proxy policy).
> **Example**: Client uses `CLUSTER KEYSLOT` to ensure multi-key operations share a slot.
> **Implication**: Enables client-side topology awareness and direct routing, reducing cross-node hops.

```python
# Section separator
pass
```

### Proxy (dmcproxy): Routing, Multiplexing, and Policies

**Role.** The RS **proxy** is the client ingress point. It calculates the target shard for a key and forwards the command, multiplexing many client connections over a small number of persistent backend connections. It is multi-threaded and designed for negligible overhead. [The Redis Enterprise Proxy (blog)](https://redis.io/blog/redis-enterprise-proxy/) • [Configure proxy policy](https://redis.io/docs/latest/operate/rs/databases/configure/proxy-policy/) • [Proxy REST object (tunable threads, dynamic scaling)](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/objects/proxy/)

**Proxy policies.**

* **single**: one active proxy serves all shards for the database.
* **all-master-shards**: one active proxy on each node hosting a primary shard.
* **all-nodes**: an active proxy on every node in the cluster.
  Policy selection controls connection fan-out and failover characteristics. [Configure proxy policy](https://redis.io/docs/latest/operate/rs/databases/configure/proxy-policy/)

```python
# Verify that the proxy routed us to a primary (master) shard
# concise one-line explanation
role = r.execute_command("ROLE")
print("role:", role[0])  # expected "master" for standard RS endpoints
```

> **Sidenote**
> [Link: Proxy REST object](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/objects/proxy/)
> **Command**: `dynamic_threads_scaling` → Automatically adjusts proxy worker threads.
> **Pattern**: Configure via REST or `rladmin tune`.
> **Example**: Increase `max_threads` to raise concurrency limits.
> **Result**: Higher throughput by scaling worker threads under load.

```python
# Section separator
pass
```

### Replication, Failover, and Rack/Zone Awareness

**Replication and failover.** With replication enabled, the dataset on each primary shard is streamed to a replica. On primary failure, RS automatically promotes the replica to primary and redirects client traffic; failover is designed to occur in seconds. [Database replication](https://redis.io/docs/latest/operate/rs/databases/durability-ha/replication/) • [Durability and HA](https://redis.io/docs/latest/operate/rs/7.8/databases/durability-ha/) • [HA technology overview](https://redis.io/technology/highly-available-redis/) • [Consistency during replication](https://redis.io/docs/latest/operate/rs/databases/durability-ha/consistency/)

**Rack/zone awareness.** RS keeps primary and replica shards on different nodes and can enforce rack/zone separation to survive localized failures. [Database replication](https://redis.io/docs/latest/operate/rs/databases/durability-ha/replication/)

**Replica HA migration.** If enabled at cluster and database levels, replica shards automatically migrate to maintain redundancy after failures. [Configure high availability for replica shards](https://redis.io/docs/latest/operate/rs/7.8/databases/configure/replica-ha/)

```python
# Basic write + read, then check TTL behavior to validate data path post-failover
# concise one-line explanation
r.execute_command("SET", "session:{a}", "v", "EX", 30)
print("get:", r.execute_command("GET", "session:{a}"))
print("ttl:", r.execute_command("TTL", "session:{a}"))
```

> **Sidenote**
> [Link: TTL](https://redis.io/docs/latest/commands/ttl/)
> **Command**: `TTL` → Returns remaining time to live (seconds) for a key.
> **Pattern**: `TTL key`
> **Example**: `TTL session:{a}`
> **Result**: `-2` if key does not exist; `-1` if key exists without expire; otherwise remaining seconds.

```python
# Section separator
pass
```

### Auto Tiering (“Big Redis”): RAM + Flash

**Concept.** **Auto Tiering** extends memory capacity by storing cold values on SSD while keeping hot metadata or values in RAM, preserving the Redis API. It significantly lowers TCO for large datasets. [Auto Tiering overview](https://redis.io/docs/latest/operate/rs/databases/auto-tiering/) • [Auto Tiering storage engines](https://redis.io/docs/latest/operate/rs/7.4/databases/auto-tiering/storage-engine/) • [Auto Tiering metrics](https://redis.io/docs/latest/operate/rs/references/metrics/auto-tiering/) • [Benchmark Auto Tiering](https://redis.io/docs/latest/operate/rs/references/memtier-benchmark/)

**Storage engines.** As of RS 7.2.4+, **Speedb** is the default recommended engine for Auto Tiering; **RocksDB** is deprecated for newer versions. [Auto Tiering storage engines](https://redis.io/docs/latest/operate/rs/7.4/databases/auto-tiering/storage-engine/)

```python
# Write a large hash to exercise value placement policies (conceptual)
# concise one-line explanation
for i in range(1000):
    r.execute_command("HSET", "catalog:{1}", f"f{i}", "x"*1024)
print("fields:", r.execute_command("HLEN", "catalog:{1}"))
```

> **Sidenote**
> [Link: Auto Tiering metrics](https://redis.io/docs/latest/operate/rs/references/metrics/auto-tiering/)
> **Concept**: `% Values in RAM` → Measures proportion of values resident in DRAM.
> **Context**: Indicates balance between RAM and flash usage per database/shard.
> **Example**: Track `Values in flash` vs `Values in RAM` to detect imbalances.
> **Implication**: Guides capacity planning and hot/cold data tuning.

```python
# Section separator
pass
```

### DNS, Endpoints, and Name Resolution

**Cluster DNS.** Each RS node runs a lightweight DNS service used for internal service discovery and endpoint resolution. DNS enables transparent failover by updating records to point to the new primary location. [Configure cluster DNS](https://redis.io/docs/latest/operate/rs/networking/cluster-dns/) • [DNS suffix REST object](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/objects/suffix/) • [Suffix requests](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/requests/suffix/)

**PowerDNS implementation.** RS bundles and manages PowerDNS internally for cluster DNS handling. [DNS setup FAQ (KB)](https://support.redislabs.com/hc/en-us/articles/26501073470866-DNS-Setup-for-Redis-Cluster-Access)

**Public and private endpoints.** Databases can expose public (external) and private FQDNs mapped to external and private IPs, respectively. [Enable private and public endpoints](https://redis.io/docs/latest/operate/rs/networking/private-public-endpoints/)

```python
# Resolve and connect via FQDN (illustrative; connection setup assumed)
# concise one-line explanation
r.execute_command("PING")
print("pong:", r.execute_command("PING"))
```

> **Sidenote**
> [Link: Configure cluster DNS](https://redis.io/docs/latest/operate/rs/networking/cluster-dns/)
> **Concept**: `DNS-managed endpoints` → RS updates DNS to steer clients to active primaries.
> **Context**: Used for automatic failover, shard migration, and endpoint moves.
> **Example**: Database FQDN resolves to the node currently hosting the target proxy.
> **Implication**: Clients connect by name; topology changes are transparent.

```python
# Section separator
pass
```

### Control Plane: Cluster Manager, State Machines, and Watchdogs

**Cluster configuration store (CCS).** RS persists cluster configuration to the **Cluster Configuration Store**; the canonical backup file is `ccs-redis.rdb`. Cluster recovery practices rely on restoring this CCS file. [Recover a failed cluster](https://redis.io/docs/latest/operate/rs/clusters/cluster-recovery/) • [rladmin cluster recover (default CCS path)](https://redis.io/docs/latest/operate/rs/references/cli-utilities/rladmin/cluster/recover/)

**State machines.** Administrative operations (e.g., shard failover/migration) execute as **state machines** tracked by the REST API. [State machine object](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/objects/state-machine/) • [Database actions (state-machine status)](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/requests/actions/bdb/) • [Actions API](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/requests/actions/)

**Watchdogs.**

* **node\_wd** monitors processes on a node (e.g., shards) and can trigger local remediation.
* **cluster\_wd** monitors node membership (gossip), drives node-level failover events, and hides master re-election details from other services.
  [Tools for identifying failures (process list references)](https://redis.io/kb/doc/2tfda6u6ew/tools-for-identifying-failures-in-redis-software/)

```python
# Observe server role and replication offsets (control-plane independent validation)
# concise one-line explanation
info_rep = r.execute_command("INFO", "replication")
print("info-replication-lines:", [line for line in info_rep.splitlines() if b"role" in line or b"offset" in line][:5])
```

> **Sidenote**
> [Link: Shard failover requests (REST)](https://redis.io/docs/latest/operate/rs/references/rest-api/requests/shards/actions/failover/)
> **Concept**: `Planned failover API` → Promotes replicas of specified shards for tests/maintenance.
> **Context**: Production failover is automatic; API is for controlled operations.
> **Example**: `POST /v1/shards/actions/failover` with shard UIDs.
> **Implication**: Validates failover runbooks without inducing faults.

```python
# Section separator
pass
```

### Provisioning and Orchestration

**cnm\_exec and orchestration.** The cluster manager applies desired state (provisioning, shard migration, listener creation, failovers) via orchestrated state machines exposed through the REST API and `rladmin`. [REST API (overview)](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/) • [rladmin (cluster/db ops)](https://redis.io/docs/latest/operate/rs/references/cli-utilities/rladmin/) • [Migrate database shards](https://redis.io/docs/latest/operate/rs/databases/migrate-shards/) • [rladmin migrate](https://redis.io/docs/latest/operate/rs/references/cli-utilities/rladmin/migrate/)

```python
# Verify key distribution across shards (when OSS Cluster API is enabled)
# concise one-line explanation
for k in [f"user:{{{i}}}:cart" for i in range(3)]:
    slot = r.execute_command("CLUSTER", "KEYSLOT", k)
    print(k, "-> slot", slot)
```

> **Sidenote**
> [Link: Migrate database shards](https://redis.io/docs/latest/operate/rs/databases/migrate-shards/)
> **Concept**: `Shard migration` → Moves shards to balance load or prepare node changes.
> **Context**: Online, automatic demotion/promotion ensures availability.
> **Example**: `rladmin migrate` or REST `/v1/bdbs/{id}/migrate`.
> **Implication**: Enables elastic rebalancing and maintenance without downtime.

```python
# Section separator
pass
```

### Active‑Active (CRDB): Geo‑Distribution

**CRDT-based multi-master.** RS **Active‑Active** databases replicate data between clusters using CRDTs, allowing concurrent writes across regions with conflict-free convergence and sub-millisecond local latency. [Active‑Active overview](https://redis.io/docs/latest/operate/rs/databases/active-active/) • [Develop with Active‑Active](https://redis.io/docs/latest/operate/rs/databases/active-active/develop/develop-for-aa/) • [`crdb-cli` (manage AA)](https://redis.io/docs/latest/operate/rs/references/cli-utilities/crdb-cli/) • [CRDB REST objects/requests](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/objects/crdb/)

**Syncer process.** Each node hosting an AA instance runs a **syncer** process that exchanges updates with peer clusters via the proxy and writes locally to primary shards. [Syncer process](https://redis.io/docs/latest/operate/rs/7.4/databases/active-active/syncer/) • [Syncer process (latest)](https://redis.io/docs/latest/operate/rs/databases/active-active/syncer/)

```python
# AA-safe write/read demo (standard Redis commands; CRDT resolution is internal)
# concise one-line explanation
r.execute_command("HSET", "order:{99}", "status", "placed")
print("status:", r.execute_command("HGET", "order:{99}", "status"))
```

> **Sidenote**
> [Link: Active‑Active geo‑distributed Redis](https://redis.io/docs/latest/operate/rs/databases/active-active/)
> **Concept**: `CRDT-based multi-master` → Concurrent writes converge without conflicts.
> **Context**: Multi-region writes, DR, and low-latency read/write locality.
> **Example**: Same key updated in two regions; CRDT merges per type semantics.
> **Implication**: Eliminates manual conflict resolution and complex client logic.

```python
# Section separator
pass
```

### APIs and Ports

**Admin/API endpoints.** RS exposes a REST API for automation; common ports include `9443` (HTTPS) and `8080` (HTTP) for the admin services, and internal service ports for nodes, databases, and metrics. [Network port configurations](https://redis.io/kb/doc/3n4mjkgfmg/what-are-network-ports-redis-software-node-needs-open/) • [REST API quick start](https://redis.io/docs/latest/operate/rs/7.4/references/rest-api/quick-start/) • [REST API (reference)](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/)

```python
# Simple health probe via Redis command path (data plane)
# concise one-line explanation
print("PING:", r.execute_command("PING"))
```

> **Sidenote**
> [Link: REST API (reference)](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/)
> **Command**: `GET /v1/actions` → Lists long-lived tasks and state machines.
> **Pattern**: `GET /v1/actions`
> **Example**: Poll to track progress of shard migrations.
> **Result**: JSON array of tasks and state-machine objects.

```python
# Section separator
pass
```

### Observability and Metrics

**Prometheus exporter.** Each cluster runs `metrics_exporter` exposing Prometheus metrics on port **8070** (and a v2 endpoint from RS 7.8.2+ under `/v2`). Metrics cover cluster, node, database, shard, and proxy. [Prometheus & Grafana with RS](https://redis.io/docs/latest/integrate/prometheus-with-redis-enterprise/) • [Observability guidance](https://redis.io/docs/latest/operate/rs/7.8/monitoring/observability/) • [Monitoring with metrics & alerts](https://redis.io/docs/latest/operate/rs/monitoring/) • [Statistics REST object](https://redis.io/docs/latest/operate/rs/references/rest-api/objects/statistics/) • [Prometheus v2 metrics preview](https://redis.io/docs/latest/integrate/prometheus-with-redis-enterprise/prometheus-metrics-definitions/)

**Stats archiver.** RS can persist CSV-formatted statistics for audit/analysis. [rladmin cluster stats\_archiver](https://redis.io/docs/latest/operate/rs/7.8/references/cli-utilities/rladmin/cluster/stats_archiver/) • [stats\_archiver REST object](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/objects/services_configuration/stats_archiver/)

```python
# Quick latency/ops insight via INFO
# concise one-line explanation
info = r.execute_command("INFO", "stats")
print("instantaneous_ops_per_sec:", [line for line in info.splitlines() if b"instantaneous_ops_per_sec" in line])
```

> **Sidenote**
> [Link: Prometheus & Grafana with RS](https://redis.io/docs/latest/integrate/prometheus-with-redis-enterprise/)
> **Concept**: `metrics_exporter` → Node-level Prometheus endpoint (default :8070).
> **Context**: External monitoring (Prometheus, Grafana, Datadog, Dynatrace).
> **Example**: Scrape `/v2` endpoint for v2 metrics (RS 7.8.2+).
> **Implication**: Unified metrics across cluster/node/db/shard/proxy.

```python
# Section separator
pass
```

### Appendix: Common Processes and Objects (Selected)

* **Proxy (dmcproxy)**: Multi-threaded client ingress and routing; tunable thread scaling via REST/`rladmin`. [Proxy REST object](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/objects/proxy/) • [Proxy policy](https://redis.io/docs/latest/operate/rs/databases/configure/proxy-policy/) • [Proxy blog](https://redis.io/blog/redis-enterprise-proxy/)
* **Shard (redis-server)**: Service unit for a hash-range in clustered databases. [Database clustering](https://redis.io/docs/latest/operate/rs/7.4/databases/configure/clustering/)
* **Cluster DNS (PowerDNS)**: Per-node DNS to resolve endpoints and orchestrate failover records. [Cluster DNS](https://redis.io/docs/latest/operate/rs/networking/cluster-dns/) • [PowerDNS (KB)](https://support.redislabs.com/hc/en-us/articles/26501073470866-DNS-Setup-for-Redis-Cluster-Access) • [Suffix object](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/objects/suffix/)
* **Cluster Configuration Store (CCS)**: Persistent cluster config database; recovery uses `ccs-redis.rdb`. [Cluster recovery](https://redis.io/docs/latest/operate/rs/clusters/cluster-recovery/) • [rladmin cluster recover](https://redis.io/docs/latest/operate/rs/references/cli-utilities/rladmin/cluster/recover/)
* **State machines**: Track and execute control-plane actions (failover, migration, etc.). [State machine object](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/objects/state-machine/) • [Actions API](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/requests/actions/)
* **Watchdogs**: `node_wd` and `cluster_wd` for local process and cluster health. [Failure-identification tools (KB)](https://redis.io/kb/doc/2tfda6u6ew/tools-for-identifying-failures-in-redis-software/)
* **Active‑Active syncer**: Per-node process for CRDB replication. [Syncer process](https://redis.io/docs/latest/operate/rs/databases/active-active/syncer/)
* **Metrics exporter**: Prometheus endpoint. [Prometheus integration](https://redis.io/docs/latest/integrate/prometheus-with-redis-enterprise/)

```python
# Section separator
pass
```

### Selected Redis Command Examples (Data Path Verification)

```python
# Write/read a key (data path)
# concise one-line explanation
r.execute_command("SET", "k1", "v1")
print("k1:", r.execute_command("GET", "k1"))
```

> **Sidenote**
> [Link: SET](https://redis.io/docs/latest/commands/set/)
> **Command**: `SET` → Assigns a value with optional flags (EX/PX/NX/XX/KEEPTTL/GET).
> **Pattern**: `SET key value [EX sec|PX ms] [NX|XX] [KEEPTTL] [GET]`
> **Example**: `SET k1 v1`
> **Result**: `OK` on success.

```python
# Expiration semantics
# concise one-line explanation
r.execute_command("EXPIRE", "k1", 10)
print("ttl:", r.execute_command("TTL", "k1"))
```

> **Sidenote**
> [Link: EXPIRE](https://redis.io/docs/latest/commands/expire/)
> **Command**: `EXPIRE` → Sets a timeout on a key (seconds).
> **Pattern**: `EXPIRE key seconds`
> **Example**: `EXPIRE k1 10`
> **Result**: `1` if timeout set, `0` otherwise.

```python
# Hash operations to validate structured writes across shards
# concise one-line explanation
r.execute_command("HSET", "user:{42}", "name", "Ada", "tier", "gold")
print("user:{42}:", r.execute_command("HGETALL", "user:{42}"))
```

> **Sidenote**
> [Link: HSET](https://redis.io/docs/latest/commands/hset/)
> **Command**: `HSET` → Sets field-value pairs in a hash.
> **Pattern**: `HSET key field value [field value ...]`
> **Example**: `HSET user:{42} name Ada`
> **Result**: Number of fields added.

```python
# Role/replication info (ensures proxy targets a primary)
# concise one-line explanation
print("ROLE:", r.execute_command("ROLE")[0])
```

> **Sidenote**
> [Link: ROLE](https://redis.io/docs/latest/commands/role/)
> **Command**: `ROLE` → Reports the instance’s role (`master`, `replica`, or `sentinel`).
> **Pattern**: `ROLE`
> **Example**: `ROLE`
> **Result**: Array with role and details (e.g., replication offsets).

```python
# Optional: hash slot check (requires OSS Cluster API)
# concise one-line explanation
print("slot:", r.execute_command("CLUSTER", "KEYSLOT", "user:{42}"))
```

> **Sidenote**
> [Link: CLUSTER KEYSLOT](https://redis.io/docs/latest/commands/cluster-keyslot/)
> **Command**: `CLUSTER KEYSLOT` → Returns the hash slot for a key.
> **Pattern**: `CLUSTER KEYSLOT key`
> **Example**: `CLUSTER KEYSLOT user:{42}`
> **Result**: Integer hash slot.

```python
# Section separator
pass
```

## Bibliography

* [Redis Cluster architecture (overview) | Docs - Redis](https://redis.io/docs/latest/operate/rs/clusters/architecture/)
* [Configure database settings | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/configure/)
* [Database clustering | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.4/databases/configure/clustering/)
* [Redis OSS Cluster API | Docs - Redis](https://redis.io/docs/latest/operate/rs/clusters/optimize/oss-cluster-api/)
* [Enable OSS Cluster API | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/configure/oss-cluster-api/)
* [Database replication | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/durability-ha/replication/)
* [Durability and high availability | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.8/databases/durability-ha/)
* [Redis High Availability (technology overview) | Redis](https://redis.io/technology/highly-available-redis/)
* [Consistency during replication | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/durability-ha/consistency/)
* [The First Rule of Redis Enterprise Proxy (blog) | Redis](https://redis.io/blog/redis-enterprise-proxy/)
* [Configure proxy policy | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/configure/proxy-policy/)
* [Proxy REST object | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/objects/proxy/)
* [Configure high availability for replica shards | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.8/databases/configure/replica-ha/)
* [Auto Tiering (overview) | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/auto-tiering/)
* [Manage Auto Tiering storage engine | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.4/databases/auto-tiering/storage-engine/)
* [Auto Tiering metrics | Docs - Redis](https://redis.io/docs/latest/operate/rs/references/metrics/auto-tiering/)
* [Benchmark an Auto Tiering enabled database | Docs - Redis](https://redis.io/docs/latest/operate/rs/references/memtier-benchmark/)
* [Configure cluster DNS | Docs - Redis](https://redis.io/docs/latest/operate/rs/networking/cluster-dns/)
* [Suffix REST object (DNS suffix) | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/objects/suffix/)
* [Suffix requests (DNS suffix) | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/requests/suffix/)
* [DNS Setup for Redis Cluster Access (PowerDNS) | Redis Support KB](https://support.redislabs.com/hc/en-us/articles/26501073470866-DNS-Setup-for-Redis-Cluster-Access)
* [Enable private and public database endpoints | Docs - Redis](https://redis.io/docs/latest/operate/rs/networking/private-public-endpoints/)
* [Recover a failed cluster | Docs - Redis](https://redis.io/docs/latest/operate/rs/clusters/cluster-recovery/)
* [rladmin cluster recover | Docs - Redis](https://redis.io/docs/latest/operate/rs/references/cli-utilities/rladmin/cluster/recover/)
* [State machine object | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/objects/state-machine/)
* [Database actions (state machines) | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/requests/actions/bdb/)
* [Actions API | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/requests/actions/)
* [Tools for identifying failures in Redis Software (process references) | Redis KB](https://redis.io/kb/doc/2tfda6u6ew/tools-for-identifying-failures-in-redis-software/)
* [Shard failover requests (REST) | Docs - Redis](https://redis.io/docs/latest/operate/rs/references/rest-api/requests/shards/actions/failover/)
* [rladmin (cluster/db operations) | Docs - Redis](https://redis.io/docs/latest/operate/rs/references/cli-utilities/rladmin/)
* [Migrate database shards | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/migrate-shards/)
* [rladmin migrate | Docs - Redis](https://redis.io/docs/latest/operate/rs/references/cli-utilities/rladmin/migrate/)
* [Active‑Active geo‑distributed Redis | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/active-active/)
* [Develop with Active‑Active | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/active-active/develop/develop-for-aa/)
* [crdb-cli | Docs - Redis](https://redis.io/docs/latest/operate/rs/references/cli-utilities/crdb-cli/)
* [CRDB object (REST) | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/objects/crdb/)
* [Syncer process (7.4) | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.4/databases/active-active/syncer/)
* [Syncer process (latest) | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/active-active/syncer/)
* [Network port configurations (admin, internal, metrics) | Redis KB](https://redis.io/kb/doc/3n4mjkgfmg/what-are-network-ports-redis-software-node-needs-open/)
* [REST API quick start | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.4/references/rest-api/quick-start/)
* [REST API (reference) | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.8/references/rest-api/)
* [Prometheus & Grafana with Redis Enterprise | Docs - Redis](https://redis.io/docs/latest/integrate/prometheus-with-redis-enterprise/)
* [Observability guidance | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.8/monitoring/observability/)
* [Monitoring with metrics & alerts | Docs - Redis](https://redis.io/docs/latest/operate/rs/monitoring/)
* [Statistics REST object | Docs - Redis](https://redis.io/docs/latest/operate/rs/references/rest-api/objects/statistics/)
* [Prometheus metrics v2 preview | Docs - Redis](https://redis.io/docs/latest/integrate/prometheus-with-redis-enterprise/prometheus-metrics-definitions/)
* [SET | Docs - Redis](https://redis.io/docs/latest/commands/set/)
* [EXPIRE | Docs - Redis](https://redis.io/docs/latest/commands/expire/)
* [TTL | Docs - Redis](https://redis.io/docs/latest/commands/ttl/)
* [HSET | Docs - Redis](https://redis.io/docs/latest/commands/hset/)
* [ROLE | Docs - Redis](https://redis.io/docs/latest/commands/role/)
* [CLUSTER KEYSLOT | Docs - Redis](https://redis.io/docs/latest/commands/cluster-keyslot/)
