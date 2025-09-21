## Redis OSS vs Enterprise: Scaling and High Availability (p. 1) :contentReference[oaicite:0]{index=0}

---

### Definitions (pp. 3–4)

#### Scalability (p. 3) :contentReference[oaicite:1]{index=1}
The capability of a system to handle increased workload by adding resources (scale-up/out). In system design, a workload is considered scalable if total throughput increases when more compute, memory, or nodes are provisioned. For formal guidance, see the *Performance Efficiency* and *Reliability* pillars, which discuss scaling patterns and trade‑offs in distributed systems ([AWS Well-Architected – Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)).  

> **Sidenote**
>
> [Link: AWS – Availability (Reliability Pillar)](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/availability.html)
>
> **Concept**: `Scalability` → Ability to increase capacity and throughput with added resources.
> **Context**: Horizontal sharding, vertical sizing, connection multiplexing, pipelining.
> **Example**: Adding shards to a Redis Cluster to increase write & read capacity.
> **Implication**: Facilitates graceful growth without redesigning core data paths.

#### High Availability (p. 4) :contentReference[oaicite:2]{index=2}
High availability (HA) focuses on maximizing uptime—ensuring the service performs its agreed function when required ([AWS – Availability](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/availability.html)). In Redis deployments, HA commonly combines replication with automatic failover and health detection.

---

### Single Redis Instance (pp. 5–6)

#### Overview (p. 5) :contentReference[oaicite:3]{index=3}
A standalone Redis process that clients access via standard SDKs.

#### Feature Summary (p. 6) :contentReference[oaicite:4]{index=4}

| Feature        | Standalone Redis |
|---|---|
| Scalability    | None |
| High-Availability | None |
| Performance    | Base |
| Multi-tenancy  | None |
| Consistency    | Relaxed (with optional replicas) |
| Operations     | Complex, manual |

*Notes (p. 6):* Simple to maintain; memory/throughput may be insufficient; read-scaling possible with replicas; no built-in sharding; `SELECT <db>` separates keyspaces but does **not** provide per-tenant config/limits/ACLs. :contentReference[oaicite:5]{index=5}

---

### Replication in Redis OSS (p. 8)

Redis OSS uses asynchronous leader–replica replication; replicas track the primary and can serve **read-only** traffic. Replication can be partially synchronized from the client using `WAIT` (replicas) and `WAITAOF` (AOF durability), which block the calling connection until conditions are met ([Redis replication](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/); [WAIT](https://redis.io/docs/latest/commands/wait/); [WAITAOF](https://redis.io/docs/latest/commands/waitaof/)). :contentReference[oaicite:6]{index=6}

```python
# Ensure a write is replicated to N replicas before proceeding (best-effort)
r.execute_command("SET", "k", "v")
ack = r.execute_command("WAIT", 1, 200)  # wait for ≥1 replica, timeout 200 ms
print(f"replicas_acked={ack}")  # e.g., 1
````

> **Sidenote**
>
> [Link: WAIT | Docs - Redis](https://redis.io/docs/latest/commands/wait/)
>
> **Command**: `WAIT` → Block until previous writes are acknowledged by replicas.
> **Pattern**: `WAIT <numreplicas> <timeout-ms>`
> **Example**: `WAIT 2 500`
> **Result**: Integer (number of replicas that acknowledged so far).

```python
# Block until this client's prior writes are fsynced to local AOF and/or replicas' AOFs
r.execute_command("SET", "k", "v")
ack = r.execute_command("WAITAOF", 1, 0, 500)  # numlocal=1, numreplicas=0, timeout=500 ms
print(f"aof_fsync_confirmations={ack}")  # e.g., [1,0]
```

> **Sidenote**
>
> [Link: WAITAOF | Docs - Redis](https://redis.io/docs/latest/commands/waitaof/)
>
> **Command**: `WAITAOF` → Block until prior writes are durably fsynced to AOF locally and/or on replicas.
> **Pattern**: `WAITAOF <numlocal> <numreplicas> <timeout-ms>`
> **Example**: `WAITAOF 1 1 1000`
> **Result**: Array of two integers `[local_fsyncs, replica_fsyncs]`.

*Clarifying context:* Replication is asynchronous by default; strong consistency is not guaranteed even when using `WAIT/WAITAOF` due to election edge cases ([Scale with Redis Cluster – consistency note](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/)).&#x20;

---

### From Replication to HA: Automatic Failover (pp. 9–11)

High availability = replication + **automatic failover**. If a primary fails, a replica should be promoted and clients should be redirected automatically.&#x20;

---

### Redis Sentinel (p. 12)

#### Role & Capabilities (p. 12)&#x20;

Sentinel processes act as watchdogs over Redis instances, notify on failures, perform **automatic failover**, and expose a **client discovery** service. Sentinel uses quorum and leader election to authorize failover ([Redis Sentinel](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/)).

```python
# Discover current master address by name via a Sentinel node
m = r.execute_command("SENTINEL", "GET-MASTER-ADDR-BY-NAME", "mymaster")
print(m)  # e.g., ["10.0.0.5", "6379"]
```

> **Sidenote**
>
> [Link: High availability with Redis Sentinel | Docs - Redis](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/)
>
> **Command**: `SENTINEL GET-MASTER-ADDR-BY-NAME` → Resolve current master endpoint.
> **Pattern**: `SENTINEL GET-MASTER-ADDR-BY-NAME <name>`
> **Example**: `SENTINEL GET-MASTER-ADDR-BY-NAME mymaster`
> **Result**: Array `[ip, port]` (current master or promoted replica).

**Sentinel Quorum & Elections.** Quorum determines how many Sentinels must agree that a master is failing; a separate **majority** of Sentinels must then elect a leader Sentinel to execute the failover ([Sentinel quorum](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/)).&#x20;

#### Feature Summary (p. 13)&#x20;

| Feature           | Sentinel                         |
| ----------------- | -------------------------------- |
| Scalability       | None                             |
| High-Availability | Yes (failovers not fast)         |
| Performance       | Base                             |
| Multi-tenancy     | None                             |
| Consistency       | Relaxed (1 master + ≥2 replicas) |
| Operations        | Complex+, manual                 |

*Notes (p. 13):* Built‑in HA; requires Sentinel-aware clients for discovery; significant operational understanding recommended.&#x20;

---

### Redis Cluster (pp. 14–28)

#### Sharding & Hash Slots (pp. 15–16)

Redis Cluster partitions the keyspace into **16,384 hash slots**. A key’s slot is `CRC16(key) mod 16384`; keys inside `{...}` use the substring (hash tags) to force co‑location ([Scale with Redis Cluster](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/); [Cluster specification](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/)).&#x20;

> **Sidenote**
>
> [Link: Scale with Redis Cluster | Docs - Redis](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/)
>
> **Concept**: `Hash slots` → Fixed 0–16383 partitioning using CRC16.
> **Context**: Determines which shard (master) owns the key; used for routing.
> **Example**: Keys `user:{42}:a` and `user:{42}:b` map to the same slot.
> **Implication**: Enables multi‑key ops within a shard; cross‑slot ops require client or proxy orchestration.

#### Cluster Topology & Client Routing (p. 21)

*Properties (from the slides):* Minimum **three master shards** (for quorum); slots are divided among shards; cluster topology can be reconfigured live (manually); clients may receive **`MOVED`** redirects when a slot migrates. Example `CLUSTER SLOTS` output:&#x20;

```
> cluster slots
1) 1) (integer) 10923
   2) (integer) 16383
   3) 1) "10.0.0.13"
      2) (integer) 7003
      3) "b93c5568..."
2) 1) (integer) 0
   2) (integer) 5460
   3) 1) "10.0.0.11"
      2) (integer) 7001
      3) "bbdd2e03..."
3) 1) (integer) 5461
   2) (integer) 10922
   3) 1) "10.0.0.12"
      2) (integer) 7002
      3) "40ee28b2..."
```

```python
# Discover slot ranges and shard endpoints
slots = r.execute_command("CLUSTER", "SLOTS")
print(len(slots) > 0)  # True

# Compute the slot for a given key (server-side)
slot = r.execute_command("CLUSTER", "KEYSLOT", "user:{42}:a")
print(f"slot={slot}")  # e.g., 7365
```

> **Sidenote**
>
> [Link: CLUSTER SLOTS | Docs - Redis](https://redis.io/docs/latest/commands/cluster-slots/)
>
> **Command**: `CLUSTER SLOTS` → Slot-to-node mapping.
> **Pattern**: `CLUSTER SLOTS`
> **Example**: `CLUSTER SLOTS`
> **Result**: Array of slot ranges and master/replica endpoints.

**Redirects.** When keys move, clients get `-MOVED` and must route to the authoritative node; during slot migration, `-ASK` is used alongside `ASKING` ([Cluster spec – MOVED/ASK](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/); [ASKING](https://redis.io/docs/latest/commands/asking/)).

```python
# When handling an -ASK, send ASKING then the original command to the target node
r.execute_command("ASKING")
val = r.execute_command("GET", "migrating:key")
print(val)
```

> **Sidenote**
>
> [Link: ASKING | Docs - Redis](https://redis.io/docs/latest/commands/asking/)
>
> **Command**: `ASKING` → Follow an `-ASK` redirection during slot migration.
> **Pattern**: `ASKING`
> **Example**: `ASKING` then `GET key`
> **Result**: `OK` (enables temporary acceptance of the redirected command).

#### Client Connections & Reads (p. 22)

Clients connect **directly** to shards. Replicas can accept **stale** reads if the client enables `READONLY` on the replica connection; otherwise, replicas redirect to masters ([READONLY](https://redis.io/docs/latest/commands/readonly/); [READWRITE](https://redis.io/docs/latest/commands/readwrite/)). Writes are asynchronous with replicas unless gated with `WAIT/WAITAOF` ([WAIT](https://redis.io/docs/latest/commands/wait/); [WAITAOF](https://redis.io/docs/latest/commands/waitaof/)).&#x20;

```python
# Enable stale reads against a replica in cluster mode
r.execute_command("READONLY")
print("OK")  # server replies OK
```

#### HA & Cluster Bus (pp. 23–25)

Redis Cluster uses a master–replica model and a **cluster bus** (binary protocol on `port + 10000`) for inter-node gossip, heartbeats (PING/PONG), and reconfiguration. Each node has a stable ID; gossip sections piggyback health and membership updates ([Cluster specification](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/)). &#x20;

> **Sidenote**
>
> [Link: Redis cluster specification | Docs - Redis](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/)
>
> **Concept**: `Cluster bus & gossip` → Mesh of nodes exchanging state and membership.
> **Context**: Health checking, slot migration, failover orchestration.
> **Example**: Gossip piggybacks node state in PING/PONG frames.
> **Implication**: Fast membership convergence and failure detection.

#### Failover Flow (p. 26)

If a master enters `FAIL`, replicas start an election; a replica that gets authorization (votes/ACKs from the **majority of masters**) is promoted. The former master, when healthy, reconfigures as a replica. (Process outline per slide; see the cluster spec’s failover and election details.)  [Redis cluster specification](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/)

#### Performance: Connections per Shard (p. 27)

Throughput per shard degrades as client connection counts grow; cluster clients must connect to **every shard**, which can raise connection overhead and routing costs (slide observation). Use client multiplexing/pipelining to amortize round‑trips ([Redis pipelining](https://redis.io/docs/latest/develop/using-commands/pipelining/)).&#x20;

#### Cluster Feature Summary (p. 28)&#x20;

| Feature           | Cluster                                |
| ----------------- | -------------------------------------- |
| Scalability       | Yes                                    |
| High-Availability | Yes                                    |
| Performance       | Potentially lower per shard (overhead) |
| Multi-tenancy     | None                                   |
| Consistency       | Relaxed (N replicas per master)        |
| Operations        | Complex++, manual                      |

*Notes (p. 28):* Built‑in HA and hash‑based sharding; **variadic** multi‑key ops are limited to keys in the same slot; requires cluster‑aware clients; non‑trivial operations (see cluster tutorial/guides).&#x20;
[Scale with Redis Cluster](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/)

---

### Redis Enterprise (pp. 29–33)

#### Architecture (pp. 30–31)

A Redis Enterprise cluster adds management and data planes that support multi‑tenancy and database isolation (per‑DB configuration, ACLs, memory limits). (Slide overview.)&#x20;

#### Enterprise Feature Summary (p. 32)&#x20;

| Feature           | Redis Enterprise         |
| ----------------- | ------------------------ |
| Scalability       | Yes, **fully automated** |
| High-Availability | Yes, **fully automated** |
| Performance       | Up to \~2× base; stable  |
| Multi-tenancy     | Yes                      |
| Consistency       | **Tunable**              |
| Operations        | Minimal, automated       |

**Operational Properties from the slides (p. 32):**

* **Automatic resharding** when increasing shard count; the database rebalances automatically.&#x20;
  *Context:* Enterprise exposes rebalancing and shard placement policies via UI/REST; shard placement ensures master/replica separation across nodes ([Shard placement policy](https://redis.io/docs/latest/operate/rs/databases/memory-performance/shard-placement-policy/); [REST – rebalance](https://redis.io/docs/latest/operate/rs/references/rest-api/requests/bdbs/actions/rebalance/)).
* **Built‑in proxy** for connection multiplexing and command re‑pipelining; per‑DB proxy abstracts shards and optimizes command fan‑out ([Configure proxy policy](https://redis.io/docs/latest/operate/rs/databases/configure/proxy-policy/); [Multiplexing explained (blog)](https://redis.io/blog/multiplexing-explained/)).&#x20;
* **Automatic failover and failure recovery** (watchdogs, supervisors).&#x20;
* **Hash‑based sharding** (supports hashtags).&#x20;
* **Variadic multi‑key support** across shards (e.g., `MSET`/`MGET`) via proxy orchestration (atomicity not guaranteed) ([KB – variadic ops in sharded DB](https://redis.io/kb/doc/2aur5jkvx8/what-needs-to-be-checked-before-resharding-a-redis-enterprise-database)).&#x20;
* **Standard clients** supported for standard clustered DBs; the proxy enables transparent routing.&#x20;
* **Node‑based quorum; 1 replica per master shard**; slide indicates replicas are not exposed for reads in this mode (policy varies by deployment).&#x20;
  *Context:* In OSS Cluster, replica reads require `READONLY`; Enterprise also supports HA replication and separate **Replica Of** read‑only databases when configured ([Enterprise replication](https://redis.io/docs/latest/operate/rs/databases/durability-ha/replication/); [Replica Of](https://redis.io/docs/latest/operate/rs/databases/import-export/replica-of/create/)).

```python
# Enterprise proxy: clients use a single endpoint; clustering is transparent.
# Example: standard multi-key op; proxy orchestrates across shards (non-atomic).
r.execute_command("MSET", "user:{42}:a", "1", "user:{99}:b", "2")
print("OK")
```

> **Sidenote**
>
> [Link: Configure proxy policy | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/configure/proxy-policy/)
>
> **Concept**: `Enterprise proxy` → Per-database, multi-threaded proxy that multiplexes connections and pipelines commands.
> **Context**: Reduces client connection fan‑out; handles re‑pipelining to shards.
> **Example**: One client connection targeting a single DB endpoint; proxy fans out to relevant shards.
> **Implication**: Lower connection overhead; enables orchestration for some cross‑slot multi‑key operations.

---

### Additional Operational Commands & Diagnostics (enriched)

```python
# Cluster health snapshot
info = r.execute_command("CLUSTER", "INFO")
print(info.splitlines()[0])  # e.g., "cluster_state:ok"

# View nodes and flags
nodes = r.execute_command("CLUSTER", "NODES")
print("master" in nodes or "myself" in nodes)

# Count keys in a slot
count = r.execute_command("CLUSTER", "COUNTKEYSINSLOT", 7365)
print(count)
```

> **Sidenote**
>
> [Link: CLUSTER INFO | Docs - Redis](https://redis.io/docs/latest/commands/cluster-info/)
>
> **Command**: `CLUSTER INFO` → Cluster status key–value report.
> **Pattern**: `CLUSTER INFO`
> **Example**: `CLUSTER INFO`
> **Result**: Text (e.g., `cluster_state:ok`, `cluster_slots_assigned:16384`).

> **Sidenote**
>
> [Link: CLUSTER NODES | Docs - Redis](https://redis.io/docs/latest/commands/cluster-nodes/)
>
> **Command**: `CLUSTER NODES` → Local node’s view of membership/slots.
> **Pattern**: `CLUSTER NODES`
> **Example**: `CLUSTER NODES`
> **Result**: Text lines (node IDs, flags, slot ranges).

---

## Tables (reconstructed)

### Feature Comparison — OSS Modes vs Enterprise (from slides; pp. 6, 13, 28, 32)

| Capability    | Standalone (OSS)   | Sentinel (OSS)             | Cluster (OSS)                                     | Enterprise              |
| ------------- | ------------------ | -------------------------- | ------------------------------------------------- | ----------------------- |
| Scalability   | None               | None                       | **Yes** (sharding)                                | **Yes (auto)**          |
| HA            | None               | **Yes** (quorum, failover) | **Yes**                                           | **Yes (auto)**          |
| Performance   | Base               | Base                       | Potentially lower per shard (connection overhead) | Up to \~2× base; stable |
| Multi‑tenancy | None               | None                       | None                                              | **Yes**                 |
| Consistency   | Relaxed (replicas) | Relaxed (1+2)              | Relaxed (N replicas)                              | **Tunable**             |
| Operations    | Manual             | Complex+, manual           | Complex++, manual                                 | Minimal, automated      |

(See slide feature summaries for exact phrasing.)   &#x20;

---

## Bibliography (order of appearance)

* [Reliability Pillar - AWS Well-Architected Framework | Docs - AWS](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)
* [Availability - Reliability Pillar | Docs - AWS](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/availability.html)
* [Redis replication | Docs - Redis](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/)
* [WAIT | Docs - Redis](https://redis.io/docs/latest/commands/wait/)
* [WAITAOF | Docs - Redis](https://redis.io/docs/latest/commands/waitaof/)
* [Scale with Redis Cluster | Docs - Redis](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/)
* [ASKING | Docs - Redis](https://redis.io/docs/latest/commands/asking/)
* [Redis cluster specification | Docs - Redis](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/)
* [Redis pipelining | Docs - Redis](https://redis.io/docs/latest/develop/using-commands/pipelining/)
* [CLUSTER SLOTS | Docs - Redis](https://redis.io/docs/latest/commands/cluster-slots/)
* [READONLY | Docs - Redis](https://redis.io/docs/latest/commands/readonly/)
* [READWRITE | Docs - Redis](https://redis.io/docs/latest/commands/readwrite/)
* [High availability with Redis Sentinel | Docs - Redis](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/)
* [Configure proxy policy | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/configure/proxy-policy/)
* [Multiplexing explained | Blog - Redis](https://redis.io/blog/multiplexing-explained/)
* [Shard placement policy | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/memory-performance/shard-placement-policy/)
* [Rebalance database requests (REST) | Docs - Redis](https://redis.io/docs/latest/operate/rs/references/rest-api/requests/bdbs/actions/rebalance/)
* [What needs to be checked before resharding a Redis Enterprise database? | KB - Redis](https://redis.io/kb/doc/2aur5jkvx8/what-needs-to-be-checked-before-resharding-a-redis-enterprise-database)
* [Enterprise database replication (HA) | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/durability-ha/replication/)
* [Create a database with Replica Of | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/import-export/replica-of/create/)
