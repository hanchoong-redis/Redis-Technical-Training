## Redis Enterprise Architecture Overview (pp. 1–20) :contentReference[oaicite:0]{index=0}

### Agenda (p. 2) :contentReference[oaicite:1]{index=1}
- Why Redis Enterprise?  
- Architecture Overview  
- Some Enterprise Features  

---

### Why Redis Enterprise? (pp. 3–4) :contentReference[oaicite:2]{index=2}
Redis Enterprise (RE) is built on Redis Open Source (Redis OSS) and augments it with operational capabilities for **real-time workloads**, **multiple data models**, and **flexible deployment (on‑prem, cloud, hybrid)**. It aims to be **the best way to manage Redis**, provide **the best operational experience**, and **reduce TCO** by simplifying scale, high availability, and manageability.

> **Sidenote**
>
> [Link: Redis Enterprise Software | Docs](https://redis.io/docs/latest/operate/rs/)  
> **Concept**: `Redis Enterprise` → A self-managed, enterprise-grade distribution of Redis OSS that adds clustering, HA, geo-distribution, observability, security, and automation.  
> **Context**: Used when teams need Redis’ speed plus enterprise guarantees (SLA-backed HA, scaling, security).  
> **Example**: Use RE to host a multi-tenant, sharded Redis with per‑DB RBAC and cross‑region replication.  
> **Implication**: Keeps Redis’ API/RESP compatibility while centralizing operations and scaling horizontally. [Redis Enterprise Software | Docs](https://redis.io/docs/latest/operate/rs/7.8/). :contentReference[oaicite:3]{index=3}

---

### Architecture Overview (p. 5) :contentReference[oaicite:4]{index=4}

#### Node (p. 6) :contentReference[oaicite:5]{index=5}
A **node** is a machine/VM/container with Redis Enterprise Software installed and joined to a cluster. Each node runs:
- **Enterprise layer** services for cluster/traffic management and monitoring.
- **OSS layer** with *N* Redis **shards** (instances) that hold data partitions.

> **Sidenote**
>
> [Link: Redis Enterprise Cluster Architecture](https://redis.io/technology/redis-enterprise-cluster-architecture/)  
> **Concept**: `Shard` → A Redis server process that stores a partition of a database’s keyspace.  
> **Context**: RE distributes a database across shards and nodes for throughput and capacity.  
> **Example**: 8-shard DB across 3 nodes; each shard handles a hash-slot range; replicas ensure HA.  
> **Implication**: Horizontal scale by adding shards/nodes; proxy abstracts placement from clients. [Redis Enterprise Cluster Architecture](https://redis.io/technology/redis-enterprise-cluster-architecture/). :contentReference[oaicite:6]{index=6}

#### Cluster (p. 7) :contentReference[oaicite:7]{index=7}
A **Redis Enterprise cluster** is a group of RE nodes with local-network characteristics that collaborate to host databases and manage shards.

- Symmetric design: each node runs the same set of RE services, can host primary or replica shards (one node is elected “master of the cluster” for control-plane tasks). (p. 8) :contentReference[oaicite:8]{index=8}  
- **Shared‑nothing** orientation: avoid shared CPU/RAM/disk/network mounts across nodes to remove single points of contention/failure. (p. 9) :contentReference[oaicite:9]{index=9}

> **Sidenote**
>
> [Link: Manage clusters | Docs](https://redis.io/docs/latest/operate/rs/7.4/clusters/)  
> **Concept**: `Cluster quorum` → Majority of nodes required for safe control-plane operations.  
> **Context**: Aids split‑brain avoidance; maintenance mode checks quorum before isolating a node.  
> **Example**: 3-node cluster tolerates a single node failure while maintaining quorum.  
> **Implication**: Prefer odd-sized clusters; RE tooling blocks maintenance if quorum would be lost. [Maintenance mode for cluster nodes | Docs](https://redis.io/docs/latest/operate/rs/clusters/maintenance-mode/). :contentReference[oaicite:10]{index=10}

#### Production Configuration for High Availability (p. 10) :contentReference[oaicite:11]{index=11}
- Use an **odd number of nodes** to reduce split‑brain risk.  
- **Quorum‑only (QO) node** can serve as the “odd” node; it does not host shards.  
- Two data nodes + one QO node suffice for HA; single-node clusters cannot enable DB replication.

> **Sidenote**
>
> [Link: `rladmin node maintenance_mode` | Docs](https://redis.io/docs/latest/operate/rs/7.8/references/cli-utilities/rladmin/node/maintenance-mode/)  
> **Concept**: `Quorum‑only node` → A node in maintenance/quorum mode that **does not** accept shards; used to preserve quorum and avoid shard placement during operations.  
> **Context**: Employed for safe maintenance, upgrades, or to provide quorum without extra capacity.  
> **Example**: Turn a node into quorum‑only before hardware service.  
> **Implication**: Helps maintain availability while preventing shard migrations into the node. :contentReference[oaicite:12]{index=12}

> **Sidenote**
>
> [Link: Azure guidance on quorum node (managed RE)](https://learn.microsoft.com/en-us/azure/azure-cache-for-redis/cache-best-practices-enterprise-tiers)  
> **Concept**: `Quorum node in managed RE` → Platform’s third node that participates in quorum decisions.  
> **Context**: Illustrates quorum patterns with RE-based managed services.  
> **Example**: Enterprise tiers run *two* data nodes plus a *quorum* node.  
> **Implication**: Confirms quorum‑centric HA design in production deployments. :contentReference[oaicite:13]{index=13}

#### Cluster Manager (p. 11) :contentReference[oaicite:14]{index=14}
Every node runs control-plane processes (the “cluster manager”): cluster health decisions, shard creation/placement/migration and re‑sharding, provisioning, resource management, and telemetry.

#### Shard (p. 12) :contentReference[oaicite:15]{index=15}
A **shard** is a Redis instance hosting a partition of a database. Each shard belongs to exactly one database and lives on exactly one node. Shards are either **primary** or **replica** for HA.

> **Sidenote**
>
> [Link: Redis replication (leader–replica) | Docs](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/)  
> **Concept**: `Primary/replica replication` → Asynchronous, low-latency replication from primary to replicas.  
> **Context**: Used per-shard for intra-cluster HA; replicas auto-reconnect and resync.  
> **Example**: Each primary shard has a replica on a different node.  
> **Implication**: Enables fast failover and read scaling; combine with `WAIT` for stronger durability on writes. [Redis replication | Docs](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/). :contentReference[oaicite:16]{index=16}

#### Database (p. 13) :contentReference[oaicite:17]{index=17}
A **database** is a container with its own endpoint, access controls, and per‑DB settings. Configurations:
- **Standalone**: one shard.  
- **HA**: one primary shard with one replica.  
- **Clustered**: data partitioned across many shards.  
- **Clustered + HA**: each primary has a replica.

| Database configuration | Shards | Replication | Notes |
|---|---:|:--:|---|
| Standalone | 1 | — | Single shard, simplest config |
| HA | 2 | ✓ | Primary + replica for availability |
| Clustered | N (N>1) | — | Partitioned across shards |
| Clustered + HA | 2N | ✓ | Each primary has a replica |

> **Sidenote**
>
> [Link: Manage databases | Docs](https://redis.io/docs/latest/operate/rs/7.8/databases/)  
> **Concept**: `Per‑DB endpoint` → Stable endpoint(s) fronted by proxies; clients don’t see shard topology.  
> **Context**: RE binds proxies to DB endpoints according to policy (single/all‑master‑shards).  
> **Example**: Switch from single proxy to “all master shards” policy for throughput.  
> **Implication**: Endpoint abstraction decouples application connectivity from shard placement. [Configure proxy policy | Docs](https://redis.io/docs/latest/operate/rs/7.8/databases/configure/proxy-policy/). :contentReference[oaicite:18]{index=18}

#### Proxy (p. 14) :contentReference[oaicite:19]{index=19}
A **DMC proxy** forwards client commands to the database’s primary shards and exposes the database endpoint(s). The slide notes C++ implementation and Memcached protocol compatibility. The proxy **abstracts distributed database complexity** and **multiple proxies can serve one endpoint**.

- Architecture: zero/near‑zero latency, multi‑threaded proxy on each node; masks shard topology changes. [Redis Enterprise Proxy (blog)](https://redis.io/blog/redis-enterprise-proxy/). :contentReference[oaicite:20]{index=20}  
- Policies: *single* proxy (default) or *all master shards*. [Configure proxy policy | Docs](https://redis.io/docs/latest/operate/rs/7.8/databases/configure/proxy-policy/). :contentReference[oaicite:21]{index=21}  
- REST control: proxies are manageable via REST (threads, restart semantics). [Proxy requests | REST API](https://redis.io/docs/latest/operate/rs/references/rest-api/requests/proxies/). :contentReference[oaicite:22]{index=22}

> **Sidenote**
>
> [Link: RESP protocol spec](https://redis.io/docs/latest/develop/reference/protocol-spec/)  
> **Concept**: `RESP/RESP3` → Wire protocol used by Redis clients and proxies.  
> **Context**: RE proxies and databases speak Redis protocol; RESP3 is a superset improving type fidelity.  
> **Example**: Arrays, maps, nulls, doubles are native in RESP3.  
> **Implication**: Broad client compatibility; easier client parsing and fewer out‑of‑band conventions. :contentReference[oaicite:23]{index=23}

---

### Some Enterprise Features (pp. 15–20) :contentReference[oaicite:24]{index=24}

#### Manageability (p. 16) :contentReference[oaicite:25]{index=25}
- Built‑in monitoring/observability endpoints & alerting.  
- Admin console (secure web UI).  
- RESTful APIs.  
- Advanced admin CLI (`rladmin`, `crdb-cli`).  
- **Kubernetes Operator** for cluster & DB lifecycle.

**REST API example (from slide):**
```bash
curl --insecure -u "user:password" https://mydb.com:9443/v1/bdbs/1
````

Authoritative references:

* **REST API** coverage (objects/requests, versions, codes). [REST API | Docs](https://redis.io/docs/latest/operate/rs/references/rest-api/). ([Redis][1])
* **Proxy REST** operations (e.g., thread tuning, restarts). [Proxy requests | Docs](https://redis.io/docs/latest/operate/rs/references/rest-api/requests/proxies/). ([Redis][2])
* **CLI utilities** (`rladmin`, `crdb-cli`, `redis-cli`, etc.). [CLI utilities | Docs](https://redis.io/docs/latest/operate/rs/7.8/references/cli-utilities/). ([Redis][3])
* **Kubernetes Operator** for RE. [Redis Enterprise for Kubernetes | Docs](https://redis.io/docs/latest/operate/kubernetes/) and [Operator architecture](https://redis.io/docs/latest/operate/kubernetes/architecture/). ([Redis][4])

> **Sidenote**
>
> [Link: `rladmin` | Docs](https://redis.io/docs/latest/operate/rs/7.8/references/cli-utilities/rladmin/)
> **Concept**: `rladmin` → Control-plane CLI for failovers, shard migrations, endpoint binding, and config.
> **Context**: Complements REST and UI; used by operators and automation.
> **Example**: `rladmin info cluster` to view proxy policies and topology.
> **Implication**: Scriptable operations and reproducible runbooks. [rladmin info | Docs](https://redis.io/docs/latest/operate/rs/references/cli-utilities/rladmin/info/). ([Redis][5])

---

### Geo‑Replication — Active‑Passive (“Replica‑Of”) (p. 17)&#x20;

**Purpose:** fast read‑only access to replicas across geographies; data migration, consolidation (multiple sources → one destination), scale‑down to fewer shards, DR/failover, and read scalability.
**Rules:** writes **only** at the source; reads at source or target; **no built‑in conflict resolution** (unidirectional).

**Enterprise implementation:** **Replica Of** (active‑passive geo distribution). [Replica Of | Docs](https://redis.io/docs/latest/operate/rs/databases/import-export/replica-of/). ([Redis][6])

> **Sidenote**
>
> [Link: REPLICAOF | Docs](https://redis.io/docs/latest/commands/replicaof/)
> **Command**: `REPLICAOF` → Configure a Redis server to replicate from a master (or stop replicating).
> **Pattern**: `REPLICAOF <host> <port>`; `REPLICAOF NO ONE`
> **Example**: `REPLICAOF 10.0.0.8 6379`
> **Result**: `OK` on success; server becomes a replica (or primary if `NO ONE`). ([Redis][7])

```python
# Configure a target instance to replicate from a source (active-passive)
r.execute_command("REPLICAOF", "10.0.0.8", "6379")
print("verification output")
```

> **Sidenote**
>
> [Link: WAIT | Docs](https://redis.io/docs/latest/commands/wait/)
> **Command**: `WAIT` → Block client until writes are acknowledged by N replicas or timeout.
> **Pattern**: `WAIT <numreplicas> <timeout_ms>`
> **Example**: `WAIT 1 1000`
> **Result**: Integer = number of replicas that acknowledged. Improves durability semantics. [Use WAIT with RE | Docs](https://redis.io/docs/latest/operate/rs/clusters/optimize/wait/). ([Redis][8])

```python
# Ensure at least one replica has acknowledged prior writes (1s timeout)
r.execute_command("WAIT", "1", "1000")
print("verification output")
```

---

### Geo‑Replication — Active‑Active (CRDB/CRDTs) (p. 18)&#x20;

**Purpose:** read **and** write the same logical dataset from multiple regions with near‑local latency.
**Mechanism:** each participating cluster hosts a **CRDB instance** (full dataset copy) and uses **CRDTs** to resolve concurrent multi‑master writes with **strong eventual consistency** (all instances converge). [Active‑Active geo‑distributed Redis | Docs](https://redis.io/docs/latest/operate/rs/databases/active-active/). ([Redis][9])

> **Sidenote**
>
> [Link: Active‑Active – Get started](https://redis.io/docs/latest/operate/rs/databases/active-active/get-started/)
> **Concept**: `CRDB instance` → A per‑cluster instance of an Active‑Active database.
> **Context**: Each instance accepts reads/writes; replication propagates via CRDT op logs/backlogs.
> **Example**: Two clusters (US/EU) with a shopping cart CRDT set; both accept writes and reconcile.
> **Implication**: Low-latency global writes while avoiding conflicts by data‑type rules. ([Redis][10])

---

### Data Tiering — Redis Auto Tiering (p. 19)&#x20;

**Goal:** near‑RAM performance for large datasets at much lower cost using **Flash (SSD/NVMe)** as an **extension of RAM** (not persistence).
**Behavior:** all **keys** are resident in RAM; **hot values** in RAM; **warm/cold values** moved to Flash by a **pluggable storage engine** (Speedb). **RAM\:Flash** ratio is configurable (≥10% RAM, per slide). Use **low‑latency NVMe** devices directly attached to the host PCIe bus.

* Official overview: RE **Auto Tiering** (DRAM+SSD with Speedb). [Auto Tiering | Docs](https://redis.io/docs/latest/operate/rs/databases/auto-tiering/). ([Redis][11])

| Tier             | Where data lives | Notes                                        |
| ---------------- | ---------------- | -------------------------------------------- |
| Keys             | DRAM             | Keys always in DRAM for O(1) metadata access |
| Hot values       | DRAM             | Lowest latency                               |
| Warm/Cold values | Flash (SSD/NVMe) | Managed by Speedb; *not* persistence         |

---

### Security — RBAC & LDAP (p. 20)&#x20;

* **RBAC**: roles provide management and/or data access permissions; associated with users, DBs, and ACLs.
* **LDAP**: authenticate/authorize via external directories (OpenLDAP, Active Directory), configurable via REST or Admin UI.

**RBAC/ACL (Redis-level)**

* Redis provides **ACLs** for command/category/keyspace permissions, users, passwords. [ACL | Docs](https://redis.io/docs/latest/operate/oss_and_stack/management/security/acl/). ([Redis][12])

> **Sidenote**
>
> [Link: ACL | Docs](https://redis.io/docs/latest/operate/oss_and_stack/management/security/acl/)
> **Command**: `ACL SETUSER` → Create/update a user with rules for auth, commands, categories, and key patterns.
> **Pattern**: `ACL SETUSER <username> on ><password> ~<pattern> +@<category> -<command>`
> **Example**: `ACL SETUSER app on >p@ss ~cache:* +@read +@write -FLUSHALL`
> **Result**: `OK` if updated/created; affects subsequent authorizations. ([Redis][12])

```python
# Create a least-privilege app user with access to cache:* and no admin ops
r.execute_command("ACL", "SETUSER", "app", "on", ">p@ss", "~cache:*", "+@read", "+@write", "-FLUSHALL")
print("verification output")
```

```python
# Inspect current users and their rules
r.execute_command("ACL", "LIST")
print("verification output")
```

**LDAP (Enterprise control plane)**

* RE integrates with LDAP for admin/user authZ, including mapping groups to roles. [LDAP authentication | Docs](https://redis.io/docs/latest/operate/rs/security/access-control/ldap/) and [Map LDAP groups to roles | Docs](https://redis.io/docs/latest/operate/rs/security/access-control/ldap/map-ldap-groups-to-roles/). ([Redis][13])

---

## Operational Commands & Diagnostics (Enriched)

> **Sidenote**
>
> [Link: INFO | Docs](https://redis.io/docs/latest/commands/info/)
> **Command**: `INFO replication` → Returns replication role and replica/primary stats.
> **Pattern**: `INFO [section]`
> **Example**: `INFO replication`
> **Result**: Bulk text with fields like `role`, `connected_slaves`, `master_link_status`. ([Redis][14])

```python
# Check replication status for a shard
r.execute_command("INFO", "replication")
print("verification output")
```

```python
# Promote a replica to primary by detaching it (stop following)
r.execute_command("REPLICAOF", "NO", "ONE")
print("verification output")
```

---

## Protocol & Compatibility (Enriched)

> **Sidenote**
>
> [Link: RESP protocol spec](https://redis.io/docs/latest/develop/reference/protocol-spec/)
> **Concept**: `RESP2/RESP3` → Redis wire protocol versions; RESP3 enriches types (maps, sets, doubles, null).
> **Context**: RE proxies and shards support standard Redis protocol, ensuring client compatibility.
> **Example**: RESP3 returns a map for `HGETALL` when requested.
> **Implication**: Better client ergonomics and introspection; same low‑latency characteristics. ([Redis][15])

> **Sidenote**
>
> [Link: Redis Enterprise ↔ Redis OSS compatibility](https://redis.io/docs/latest/operate/rs/7.8/references/compatibility/)
> **Concept**: `Compatibility` → RE aims for full Redis OSS command/protocol compatibility; supports RESP2/RESP3 and (optionally) OSS Cluster API.
> **Context**: Lets you use standard Redis clients and modules across RE.
> **Example**: Toggle OSS Cluster API when clients expect multiple endpoints instead of a single proxy endpoint.
> **Implication**: Smooth migrations from OSS; predictable API behavior. ([Redis][16])

---

## Bibliography (order of appearance)

* [Redis Enterprise Software | Docs – Operate](https://redis.io/docs/latest/operate/rs/)
* [Redis Enterprise Software v7.8 overview | Docs](https://redis.io/docs/latest/operate/rs/7.8/)
* [Redis Enterprise Cluster Architecture | Docs – Redis](https://redis.io/technology/redis-enterprise-cluster-architecture/)
* [Manage clusters | Docs – Redis Enterprise Software](https://redis.io/docs/latest/operate/rs/7.4/clusters/)
* [Maintenance mode for cluster nodes | Docs – Redis Enterprise Software](https://redis.io/docs/latest/operate/rs/clusters/maintenance-mode/)
* [rladmin node maintenance\_mode | Docs – Redis Enterprise Software](https://redis.io/docs/latest/operate/rs/7.8/references/cli-utilities/rladmin/node/maintenance-mode/)
* [Best practices for Enterprise tiers (quorum node) | Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-cache-for-redis/cache-best-practices-enterprise-tiers)
* [Redis replication | Docs – Redis OSS](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/)
* [Manage databases | Docs – Redis Enterprise Software](https://redis.io/docs/latest/operate/rs/7.8/databases/)
* [Configure proxy policy | Docs – Redis Enterprise Software](https://redis.io/docs/latest/operate/rs/7.8/databases/configure/proxy-policy/)
* [Redis Enterprise Proxy (architecture) | Redis Blog](https://redis.io/blog/redis-enterprise-proxy/)
* [Proxy requests (REST) | Docs – Redis Enterprise Software](https://redis.io/docs/latest/operate/rs/references/rest-api/requests/proxies/)
* [RESP protocol specification | Docs – Redis](https://redis.io/docs/latest/develop/reference/protocol-spec/)
* [REST API (overview) | Docs – Redis Enterprise Software](https://redis.io/docs/latest/operate/rs/references/rest-api/)
* [CLI utilities reference | Docs – Redis Enterprise Software](https://redis.io/docs/latest/operate/rs/7.8/references/cli-utilities/)
* [Redis Enterprise for Kubernetes | Docs](https://redis.io/docs/latest/operate/kubernetes/)
* [Redis Enterprise for Kubernetes — Architecture | Docs](https://redis.io/docs/latest/operate/kubernetes/architecture/)
* [Replica Of (Active‑Passive geo) | Docs – Redis Enterprise Software](https://redis.io/docs/latest/operate/rs/databases/import-export/replica-of/)
* [REPLICAOF | Docs – Redis Commands](https://redis.io/docs/latest/commands/replicaof/)
* [WAIT | Docs – Redis Commands](https://redis.io/docs/latest/commands/wait/)
* [Use WAIT to improve durability (RE) | Docs](https://redis.io/docs/latest/operate/rs/clusters/optimize/wait/)
* [Active‑Active geo‑distributed Redis | Docs](https://redis.io/docs/latest/operate/rs/databases/active-active/)
* [Active‑Active – Get started | Docs](https://redis.io/docs/latest/operate/rs/databases/active-active/get-started/)
* [Auto Tiering | Docs – Redis Enterprise Software](https://redis.io/docs/latest/operate/rs/databases/auto-tiering/)
* [ACL (Access Control Lists) | Docs – Redis](https://redis.io/docs/latest/operate/oss_and_stack/management/security/acl/)
* [LDAP authentication | Docs – Redis Enterprise Software](https://redis.io/docs/latest/operate/rs/security/access-control/ldap/)
* [Map LDAP groups to roles | Docs – Redis Enterprise Software](https://redis.io/docs/latest/operate/rs/security/access-control/ldap/map-ldap-groups-to-roles/)


[1]: https://redis.io/docs/latest/operate/rs/references/rest-api/?utm_source=chatgpt.com "REST API | Docs - Redis"
[2]: https://redis.io/docs/latest/operate/rs/references/rest-api/requests/proxies/?utm_source=chatgpt.com "Proxy requests | Docs - Redis"
[3]: https://redis.io/docs/latest/operate/rs/7.8/references/cli-utilities/?utm_source=chatgpt.com "Command-line utilities | Docs - Redis"
[4]: https://redis.io/docs/latest/operate/kubernetes/?utm_source=chatgpt.com "Redis Enterprise for Kubernetes | Docs"
[5]: https://redis.io/docs/latest/operate/rs/7.8/references/cli-utilities/rladmin/?utm_source=chatgpt.com "rladmin | Docs - Redis"
[6]: https://redis.io/docs/latest/operate/rs/databases/import-export/replica-of/?utm_source=chatgpt.com "Replica Of geo-distributed Redis | Docs"
[7]: https://redis.io/docs/latest/commands/replicaof/?utm_source=chatgpt.com "REPLICAOF | Docs - Redis"
[8]: https://redis.io/docs/latest/commands/wait/?utm_source=chatgpt.com "WAIT | Docs - Redis"
[9]: https://redis.io/docs/latest/operate/rs/databases/active-active/?utm_source=chatgpt.com "Active-Active geo-distributed Redis | Docs"
[10]: https://redis.io/docs/latest/operate/rs/databases/active-active/get-started/?utm_source=chatgpt.com "Get started with Redis Enterprise Active-Active databases"
[11]: https://redis.io/docs/latest/operate/rs/databases/auto-tiering/?utm_source=chatgpt.com "Auto Tiering | Docs - Redis"
[12]: https://redis.io/docs/latest/operate/oss_and_stack/management/security/acl/?utm_source=chatgpt.com "ACL | Docs - Redis"
[13]: https://redis.io/docs/latest/operate/rs/security/access-control/ldap/?utm_source=chatgpt.com "LDAP authentication | Docs - Redis"
[14]: https://redis.io/docs/latest/commands/info/?utm_source=chatgpt.com "INFO | Docs - Redis"
[15]: https://redis.io/docs/latest/develop/reference/protocol-spec/?utm_source=chatgpt.com "Redis serialization protocol specification | Docs"
[16]: https://redis.io/docs/latest/operate/rs/7.8/references/compatibility/?utm_source=chatgpt.com "Redis Enterprise compatibility with Redis Open Source | Docs"
