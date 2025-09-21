## Deeper into Redis Enterprise Architecture (pp. 1–24)

### Agenda (p. 2)
- Node processes overview  
- Cluster configuration  
- Provisioning: CNM and State Machines  
- Other processes  
- Service Manager  
- Improvements over Redis OSS  

---

### Node Processes (pp. 3–5)

#### What runs on a node? (p. 4)
Redis Enterprise nodes run multiple categories of processes:

- **APIs & UIs**:  
  - `envoy_control_plane`, `envoy`  
  - `cnm_http`, `cm_server`  
  - `crdb_coordinator`, `metrics_exporter`

- **Management**:  
  - `redis_mgr`, `info_provider`, `resource_mgr`  
  - `alerts_mgr`, `backup_mgr`, `stats_archiver`, `job_scheduler`  
  - `node_wd`

- **Clustering**:  
  - `ccs`, `cluster_wd`, `heartbeatd`, `gossip_envoy`

- **Provisioning**:  
  - `cnm_exec`, `crdb_worker`

- **Data Path**:  
  - `dmcproxy`, `syncer`

- **Discovery Services**:  
  - `sentinel_service`, `mdns_server`, `pdns_server`, `pdns_ccs backend`

#### Service Management with `supervisorctl` (p. 5)
`supervisorctl` is used to manage node services:

```bash
supervisorctl status        # list running services
supervisorctl stop all      # stop all services
supervisorctl start all     # start all services
supervisorctl restart all   # restart all services
supervisorctl stop heartbeatd cluster_wd  # stop selected services
````

**Example:**

```bash
# supervisorctl status
alert_mgr          RUNNING pid 1410, uptime 1 day, 1:28:49
ccs                RUNNING pid 1373, uptime 1 day, 1:28:50
cluster_wd         RUNNING pid 1377, uptime 1 day, 1:28:50
cm_server          RUNNING pid 1411, uptime 1 day, 1:28:49
```

> **Sidenote**
> [Supervisorctl | Supervisor Docs](http://supervisord.org/)
> **Command**: `supervisorctl` → Controls processes under `supervisord`.
> **Pattern**: `supervisorctl [options] action [args]`
> **Example**: `supervisorctl restart cm_server`
> **Result**: Restarts the given process.

---

### Cluster Configuration (pp. 6–9)

#### Cluster Configuration Store (CCS) (p. 7)

* Stores **cluster metadata**: databases, nodes, endpoints, statistics, state machine status.
* Accessed only via the **Master of the Cluster** (single source of truth).
* Replicated on all nodes for **HA**.
* Implemented as a **Redis database**, accessed via Redis clients.
* Uses **Pub/Sub channels** to notify clients of changes.
* Accessible via CLI: `ccs-cli`.

> **Sidenote**
> [Pub/Sub | Docs - Redis](https://redis.io/docs/latest/develop/interact/pubsub/)
> **Concept**: Publish/Subscribe → Message distribution pattern where publishers send messages to channels and subscribers receive updates.
> **Context**: CCS uses it to propagate configuration changes.
> **Example**: `SUBSCRIBE cluster-updates`
> **Implication**: Ensures consistency across all nodes.

#### Cluster watchdog, CCS and heartbeatd (pp. 8–9)

* **Cluster Watchdog (`cluster_wd`)**:

  * Elects the **Master node**.
  * Uses **Gossip Protocol** for node state sharing.
  * Reconfigures CCS replicas on master change.

* **Heartbeat Daemon (`heartbeatd`)**:

  * Provides **Port Forwarder services** to redirect CCS queries to master.
  * Tracks node liveness, writing `node_heartbeat:N` keys in CCS with TTL.
  * If key exists → node is alive.

---

### Internode Encryption (INE) (pp. 10–12)

#### Control Plane INE (CPINE) (p. 11)

* Enabled by default.
* Encrypts: gossip, `heartbeatd`, CCS replication.

#### Data Path INE (DPINE) (p. 12)

* Disabled by default for performance reasons.
* Encrypts:

  * DMC proxy ↔ shards
  * Shard replication
  * Active-Active synchronization

> **Sidenote**
> [TLS | Docs - Redis](https://redis.io/docs/latest/operate/oss_and_stack/stack-with-enterprise/security/encryption/tls/)
> **Concept**: Internode Encryption → Protects cluster communications with TLS.
> **Context**: Meets compliance requirements (e.g., HIPAA, PCI-DSS).
> **Example**: Configuring Redis Enterprise to enable INE.
> **Implication**: Ensures secure transport without altering in-node communication.

---

### Provisioning (pp. 13–15)

#### Cluster and Node Manager (CNM) (p. 14)

* `cnm_exec` executes **state machines** for cluster changes.
* Instructions received from CCS via Pub/Sub queues.
* Tasks: provisioning, shard migration, resharding, failovers.
* Status tracked in CCS.

#### More on CNM (p. 15)

* `cnm_http`: Receives REST API requests (create DB, add node, etc.).
* `cnm_exec`:

  * On master: manages state machines.
  * On replicas: executes requests for local resources.
* `crdb_worker`: Similar to `cnm_exec`, but for **Active-Active CRDBs**.

**Database creation example flow**:
REST request → `cnm_http` → CCS → `cnm_exec` (master) → cluster processes.

---

### Other Notable Processes (pp. 16–19)

#### Management (p. 17)

| Process            | Description                                             |
| ------------------ | ------------------------------------------------------- |
| `redis_mgr`        | Manages local shards (memory, statistics, AOF rewrites) |
| `rl_info_provider` | Samples shard INFO, exposes via JSON-RPC                |
| `resource_mgr`     | Collects metrics, triggers eviction/slave migration     |
| `alert_mgr`        | Triggers alerts                                         |
| `backup_mgr`       | Manages backups                                         |
| `stats_archiver`   | Stores detailed statistics                              |
| `job_scheduler`    | Schedules jobs: CCS snapshots, backups, rotation        |
| `node_wd`          | Monitors node health                                    |

#### API / UI (p. 18)

* `envoy`: high-performance proxy.
* `cnm_http`: API server.
* `cm_server`: Web UI.
* `crdb_coordinator`: CRDB management.
* `metrics_exporter`: Prometheus endpoint.

#### Discovery Services (p. 19)

* `pdns_server`: Authoritative DNS server for BDBs/nodes.
* `mdns_server`: Handles `.local` DNS (testing only).
* `sentinel`: Redis Sentinel protocol for discovery.

#### Data Path (p. 19)

* `dmcproxy`: Handles data path proxying.
* `syncer`: Syncs data across replicas.

---

### Improvements over Redis OSS (pp. 20–23)

#### Data Persistence (pp. 21–22)

* **Redis OSS**:

  * Persistence slower, direct disk writes.

* **Redis Enterprise**:

  * Replica shards act as **disk I/O threads**.
  * Disk latency doesn’t impact DB performance.
  * Enhancements: optimal AOF rewrite, chunk sizes.
  * Supports fsync modes (`everysec`, `always`).

```python
# Using WAIT to check durability
r.execute_command("WAIT", "2", "1000")
print("Number of replicas that acknowledged write")
```

> **Sidenote**
> [WAIT | Docs - Redis](https://redis.io/docs/latest/commands/wait/)
> **Command**: `WAIT` → Ensures write acknowledgment from replicas.
> **Pattern**: `WAIT numreplicas timeout`
> **Example**: `WAIT 2 1000`
> **Result**: Number of replicas that confirmed write within timeout.

#### Diskless Synchronization (p. 23)

* **Redis OSS**:

  * Default: disk-based initial sync.
  * Diskless sync (master) since v2.8.18 (default since v7.0).
  * Diskless load (replica) since v6.0, but not default.

* **Redis Enterprise**:

  * Diskless sync/load by default.
  * Custom implementation for faster recovery.

---

## Bibliography

* [Supervisorctl | Supervisor Docs](http://supervisord.org/)
* [Pub/Sub | Docs - Redis](https://redis.io/docs/latest/develop/interact/pubsub/)
* [TLS | Docs - Redis](https://redis.io/docs/latest/operate/oss_and_stack/stack-with-enterprise/security/encryption/tls/)
* [WAIT | Docs - Redis](https://redis.io/docs/latest/commands/wait/)
