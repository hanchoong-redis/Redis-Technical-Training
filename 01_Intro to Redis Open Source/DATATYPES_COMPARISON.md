# Redis Data Types – One‑Stop Expert Reference (Redis OSS 7.x/8.x)

## Core Data Structures (updated)

| **Data Structure**         | **Concept & Behavior**                                     | **Encodings (Redis ≥ 7)**                                                                 | **Sizing / Limits**                                            | **Use Cases (3)**                                              | **What Distinguishes It**                                        | **Best At / Why**                                                                            |
| -------------------------- | ---------------------------------------------------------- | ----------------------------------------------------------------------------------------- | -------------------------------------------------------------- | -------------------------------------------------------------- | ---------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| **Strings**                | Binary‑safe byte sequences; counters, bit ops, substrings. | `int`, `embstr` (≤44 B), `raw`.                                                           | Value up to **512 MB**; key name up to **512 MB**.             | 1) Tokens/configs  2) Counters/rate limits  3) Flags/bitfields | Small‑object encodings, atomic `INCR*`, rich bit ops.            | **Simple values & counters** — minimal overhead; atomic ops. ([Redis][1])                    |
| **Lists**                  | Ordered sequences; fast pushes/pops at ends.               | `quicklist` (linked nodes of listpacks).                                                  | Up to **2^32‑1** elements; each element up to 512 MB.          | 1) Queues/stacks  2) Activity feeds  3) Job dispatch           | Blocking pops/moves; trim & range ops.                           | **Queues & logs** — cheap FIFO/LIFO ends. ([Redis][2])                                       |
| **Sets**                   | Unordered unique strings.                                  | `intset` (small integer sets), `listpack` (small non‑int sets, Redis ≥ 7.2), `hashtable`. | Up to **2^32‑1** members.                                      | 1) Unique users  2) Tag lookups  3) Recommendations            | Uniqueness + set algebra.                                        | **Membership & uniqueness** — O(1) checks; algebra ops. ([Redis][2])                         |
| **Hashes**                 | Field→value maps under one key.                            | `listpack` (small) or `hashtable`.                                                        | Up to **2^32‑1** fields; per‑field TTL available (Redis 7.4+). | 1) User profiles  2) Config maps  3) Session state             | Compact “object” representation; per‑field ops.                  | **Structured objects** — memory‑efficient; per‑field updates. ([Redis][2])                   |
| **Sorted Sets (Zsets)**    | Unique members ordered by floating‑point score.            | `listpack` (small) or **skiplist+dict** (normal).                                         | Up to **2^32‑1** members; scores are doubles.                  | 1) Leaderboards  2) Priority queues  3) Time‑indexed index     | Sorted order + range queries + rank ops.                         | **Ranked data** — ordering & range queries. ([Redis][2])                                     |
| **Streams**                | Append‑only log with monotonic IDs; consumer groups.       | **Radix tree (`rax`) of listpacks**.                                                      | Unbounded unless trimmed (`XTRIM`/`MAXLEN`/`MINID`).           | 1) Event logs  2) Messaging  3) Pipelines                      | Replayable log; consumer groups, ack & claim.                    | **Event‑driven systems** — persistence, replay, groups. ([Redis][2])                         |
| **Bitmaps** *(on Strings)* | Treat Strings as bit arrays; bitwise ops & counters.       | String encodings (above).                                                                 | \~**2^32** flags per 512 MB string.                            | 1) Activity tracking  2) Feature flags  3) Presence checks     | Ultra‑dense boolean storage.                                     | **Binary states at scale** — memory‑efficient, fast bit ops. ([Redis][3])                    |
| **HyperLogLogs**           | Probabilistic distinct counter.                            | HLL internal registers.                                                                   | **\~12 KB** fixed; **\~0.81%** std. error.                     | 1) UV counts  2) Analytics  3) Massive dedup est.              | Constant memory (mergeable) regardless of N.                     | **Huge unique counts** — tiny memory for estimates. ([Redis][4])                             |
| **Geospatial Indexes**     | Store lon/lat; radius/box queries.                         | Backed by **Sorted Set**; score stores geohash‑derived value.                             | Zset limits; double precision.                                 | 1) Nearby search  2) Distance calc  3) Proximity recs          | Specialized `GEO*` ops; `GEOSEARCH` (6.2+) replaces `GEORADIUS`. | **Location queries** — efficient radius & box searches. ([redis-doc-test.readthedocs.io][5]) |

*Encoding note:* Redis ≥ 7 uses **listpack** for small lists/hashes/zsets; Redis ≥ 7.2 adds listpack for small non‑integer sets. Lists are `quicklist` nodes that contain listpacks. ([Redis][2])

## Big‑O Quick Reference (typical operations)

| Type              | Common Ops → Complexity                                                                                                                                                                                           |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Strings**       | `GET`/`SET` **O(1)**; `INCR*` **O(1)**; `GETRANGE` **O(n)** over returned slice; `SETRANGE` amortized **O(1)** / **O(m)** by value; `BITCOUNT`/`BITOP` **O(n)**; `BITFIELD` **O(1)** per subcommand. ([Redis][6]) |
| **Lists**         | `LPUSH`/`RPUSH`/`LPOP`/`RPOP` **O(1)**; `LRANGE` **O(S+N)**; `LINDEX`/`LREM`/`LINSERT` **O(n)**; blocking variants wait. ([Redis][7])                                                                             |
| **Sets**          | `SADD`/`SREM`/`SISMEMBER` **O(1)**; `SMEMBERS` **O(n)**; `SINTER`/`SUNION`/`SDIFF` proportional to total input; `SINTERCARD` returns only intersection cardinality efficiently. ([Redis][8])                      |
| **Hashes**        | `HSET`/`HGET` **O(1)**; `HDEL` **O(1)**; `HGETALL`/`HKEYS`/`HVALS` **O(n)**; field TTL ops `HEXPIRE*` **O(k)** over specified fields. ([Redis][9])                                                                |
| **Sorted Sets**   | `ZADD`/`ZREM` **O(log N)**; `ZRANGE*`/`ZREVRANGE*` **O(log N + M)**; `ZRANK`/`ZSCORE` **O(log N)/O(1)**; set algebra (`ZUNION`/`ZINTER`) depends on inputs; `ZINTERCARD` for counts. ([Redis][10])                |
| **Streams**       | `XADD` amortized **O(1)**; `XRANGE`/`XREAD` cost \~returned items; group ops (`XACK`/`XAUTOCLAIM`) depend on pending size. ([Redis][11])                                                                          |
| **Bitmaps**       | `SETBIT`/`GETBIT` **O(1)**; `BITCOUNT`/`BITOP` **O(n)**. ([Redis][12])                                                                                                                                            |
| **HyperLogLogs**  | `PFADD`/`PFCOUNT` **O(1)** typical; `PFMERGE` proportional to registers (still small). ([Redis][13])                                                                                                              |
| **Geospatial**    | `GEOSEARCH` time \~ area coverage; `GEODIST` **O(1)** per pair. `GEORADIUS*` **deprecated**. ([Redis][14])                                                                                                        |
| **Keyspace/Scan** | `SCAN` family provides incremental iteration: each call **O(1)** (total work **O(N)** over collection). Avoid blocking `KEYS` in prod. ([Redis][15])                                                              |

## Commands Cheat Sheet (by type)

* **Strings:** `SET` (`EX|PX|NX|XX|GET`), `GET`, `MGET`, `INCR|DECR|*BY`, `APPEND`, `GETRANGE`/`SETRANGE`, `GETEX`, `SETEX`, `SETBIT`/`GETBIT`/`BITCOUNT`/`BITOP`, `BITFIELD`/`BITFIELD_RO`. ([Redis][16])
* **Lists:** `LPUSH|RPUSH|LPUSHX|RPUSHX`, `LPOP|RPOP|BLPOP|BRPOP`, `LMOVE|BLMOVE|LMPOP|BLMPOP`, `LRANGE`, `LTRIM`, `LINDEX`, `LSET`, `LREM`. ([Redis][17])
* **Sets:** `SADD|SREM`, `SISMEMBER|SMISMEMBER`, `SCARD`, `SMEMBERS|SSCAN`, `SPOP|SRANDMEMBER`, `SINTER|SUNION|SDIFF`, `SINTERCARD`, `S*STORE`. ([Redis][8])
* **Hashes:** `HSET|HSETNX|HGET|HMGET`, `HDEL`, `HINCRBY|HINCRBYFLOAT`, `HGETALL|HKEYS|HVALS|HSCAN`, **field TTL (Redis 7.4+):** `HEXPIRE|HPEXPIRE|HEXPIREAT|HPEXPIREAT|HSETEX|HGETEX|HPERSIST|HTTL|HPTTL|HEXPIRETIME|HPEXPIRETIME`. ([Redis][18])
* **Sorted Sets:** `ZADD|ZINCRBY`, `ZREM`, `ZSCORE|ZMSCORE`, `ZCARD|ZCOUNT`, `ZRANGE|ZRANGEBYSCORE|ZRANGEBYLEX|ZRANK`, `ZPOPMIN|ZPOPMAX|ZMPOP`, `ZUNION|ZINTER|ZDIFF`, `ZINTERCARD`. ([Redis][10])
* **Streams:** `XADD` (`MAXLEN|MINID`), `XRANGE|XREVRANGE`, `XREAD`, `XGROUP` (`CREATE|SETID|DESTROY|...`), `XREADGROUP`, `XACK|XPENDING|XAUTOCLAIM|XCLAIM`, `XTRIM`, `XLEN`. ([Redis][11])
* **Bitmaps:** `SETBIT|GETBIT|BITCOUNT|BITOP`, `BITFIELD|BITFIELD_RO`. ([Redis][16])
* **HyperLogLogs:** `PFADD|PFCOUNT|PFMERGE`. ([Redis][13])
* **Geospatial:** `GEOADD`, `GEOSEARCH|GEOSEARCHSTORE`, `GEODIST`, `GEOPOS`, (`GEORADIUS*` **deprecated**). ([Redis][14])

## Encodings & Tunables (memory‑efficiency thresholds)

| **Type**   | **Space‑saving encoding**                                                   | **Key config (defaults)**                                                                                                  |
| ---------- | --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| Hash       | `listpack` for small hashes                                                 | `hash-max-listpack-entries` **512**; `hash-max-listpack-value` **64** bytes                                                |
| Sorted Set | `listpack` for small zsets                                                  | `zset-max-listpack-entries` **128**; `zset-max-listpack-value` **64** bytes                                                |
| Set        | `intset` (all‑ints small set); `listpack` (small non‑int sets, Redis ≥ 7.2) | `set-max-intset-entries` **512**; *(Redis ≥ 7.2 adds)* `set-max-listpack-entries` **128**, `set-max-listpack-value` **64** |
| List       | `quicklist` nodes store listpacks (compact chunks)                          | see `list-max-listpack-size` (in `redis.conf`)                                                                             |

Space‑saving encodings auto‑convert to general encodings when thresholds are exceeded. ([Redis][19])

## Cross‑Cutting Semantics & Limits

* **Key & element limits:** max key name **512 MB**; String value **512 MB** (therefore \~**2^32** bits for bitmaps). Aggregates (list/hash/set/zset) can hold **2^32‑1** elements. ([Redis][20])
* **Expiration:** Key‑level TTL: `EXPIRE|PEXPIRE|EXPIREAT|PERSIST|TTL|PTTL`. Hash **field‑level TTL** (Redis 7.4+): `HEXPIRE|HPEXPIRE|HEXPIREAT|HPEXPIREAT|HPERSIST|HTTL|HPTTL|HEXPIRETIME|HPEXPIRETIME`; `HSETEX|HGETEX`. ([Redis][18])
* **Inspection & memory:** `TYPE key`, `OBJECT ENCODING key`, `MEMORY USAGE key`. Encodings include `embstr`/`raw`/`int` (strings), `quicklist` (lists), `listpack` (small aggregates), `skiplist` (zsets), `stream` (rax of listpacks). ([Redis][2])
* **Cluster multi‑key ops:** Multi‑key ops require keys in the same **hash slot**; use **hash tags** `{…}` in key names to co‑locate related keys. ([Redis][21])

## Notes & Version Highlights

* **Geospatial:** Use `GEOSEARCH`/`GEOSEARCHSTORE`; `GEORADIUS*` is **deprecated** since Redis 6.2. Implementation stores **geohash‑derived** score in a zset and uses Haversine for distances. ([Redis][14])
* **Streams internals:** Streams are implemented as a **radix tree of listpacks**; `XADD` is amortized **O(1)**; consumer groups provide ack/pending/claim semantics. ([Redis][2])
* **HyperLogLog:** Fixed memory (\~12 KB) with \~0.81% standard error; ideal for approximate cardinality. ([Redis][4])

## Quick Examples (patterns by type)

* **Counters (Strings):** `SET key 0 NX` → `INCRBY key 1`.
* **FIFO queue (Lists):** producers `RPUSH q item`; consumers `BLPOP q timeout`.
* **Membership (Sets):** `SADD users u42`; `SISMEMBER users u42`.
* **Object (Hashes):** `HSET user:1 name Alice plan pro`; *(field TTL)* `HEXPIRE user:1 3600 FIELDS 1 plan`.
* **Leaderboard (Zset):** `ZADD board 1337 user:1`; `ZREVRANGE board 0 9 WITHSCORES`.
* **Event log (Streams):** `XADD events * type signup uid 1`; `XREADGROUP GROUP g c COUNT 100 STREAMS events >`.
* **Bitmap active‑day flag:** `SETBIT act:2025‑09‑21 42 1`; `BITCOUNT act:2025‑09‑21`.
* **HLL uniques:** `PFADD u:page /home ip:1.2.3.4`; `PFCOUNT u:page`.
* **Geo nearby:** `GEOADD shops -122.42 37.77 "SF"`; `GEOSEARCH shops FROMLONLAT -122.4 37.8 BYRADIUS 5 km WITHDIST`. ([Redis][18])

---

*This reference reflects Redis OSS 7.x/8.x behavior and encodings; see linked command and docs pages for precise semantics and updates.*

* [OBJECT ENCODING | Docs - Redis](https://redis.io/docs/latest/commands/object-encoding/?utm_source=chatgpt.com)
* [OBJECT ENCODING | Docs](https://redis.io/docs/latest/commands/object-encoding/)
* [Redis bitmaps | Docs](https://redis.io/docs/latest/develop/data-types/bitmaps/?utm_source=chatgpt.com)
* [HyperLogLog | Docs - Redis](https://redis.io/docs/latest/develop/data-types/probabilistic/hyperloglogs/?utm_source=chatgpt.com)
* [Geoadd - Redis Documentation - Read the Docs](https://redis-doc-test.readthedocs.io/en/latest/commands/geoadd/?utm_source=chatgpt.com)
* [GETRANGE | Docs - Redis](https://redis.io/docs/latest/commands/getrange/?utm_source=chatgpt.com)
* [LRANGE | Docs - Redis](https://redis.io/docs/latest/commands/lrange/?utm_source=chatgpt.com)
* [SADD | Docs - Redis](https://redis.io/docs/latest/commands/sadd/?utm_source=chatgpt.com)
* [HSET | Docs - Redis](https://redis.io/docs/latest/commands/hset/?utm_source=chatgpt.com)
* [ZADD | Docs - Redis](https://redis.io/docs/latest/commands/zadd/?utm_source=chatgpt.com)
* [Redis Streams | Docs](https://redis.io/docs/latest/develop/data-types/streams/?utm_source=chatgpt.com)
* [BITCOUNT | Docs - Redis](https://redis.io/docs/latest/commands/bitcount/?utm_source=chatgpt.com)
* [PFADD | Docs - Redis](https://redis.io/docs/latest/commands/pfadd/?utm_source=chatgpt.com)
* [GEOSEARCH | Docs - Redis](https://redis.io/docs/latest/commands/geosearch/?utm_source=chatgpt.com)
* [SCAN | Docs - Redis](https://redis.io/docs/latest/commands/scan/?utm_source=chatgpt.com)
* [BITFIELD | Docs - Redis](https://redis.io/docs/latest/commands/bitfield/?utm_source=chatgpt.com)
* [Redis lists | Docs](https://redis.io/docs/latest/develop/data-types/lists/?utm_source=chatgpt.com)
* [HEXPIRE | Docs](https://redis.io/docs/latest/commands/hexpire/)
* [Memory optimization | Docs - Redis](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/memory-optimization/?utm_source=chatgpt.com)
* [Keys and values | Docs - Redis](https://redis.io/docs/latest/develop/using-commands/keyspace/?utm_source=chatgpt.com)
* [Scale with Redis Cluster | Docs](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/?utm_source=chatgpt.com)
