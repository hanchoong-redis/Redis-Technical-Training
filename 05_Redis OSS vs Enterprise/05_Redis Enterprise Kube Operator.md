````markdown
## Redis Enterprise Kubernetes Operator – [Link to Video](https://www.youtube.com/watch?v=7UBlNsyHSQA)

### Overview

Kubernetes uses controllers (control loops) that continually compare the observed cluster state with the desired state and act to reconcile differences. Custom resources and controllers extend this model to domain-specific operations. The Redis Enterprise Operator implements this pattern to manage Redis Enterprise clusters and databases on Kubernetes. [Controllers | Kubernetes](https://kubernetes.io/docs/concepts/architecture/controller/), [Custom Resources | Kubernetes](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/), [Extend the Kubernetes API with CRDs | Kubernetes](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/), [Operator-based architecture | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/7.4.6/architecture/operator/)

> **Sidenote**  
>
> [Link: Controllers | Kubernetes](https://kubernetes.io/docs/concepts/architecture/controller/)  
>
> **Concept**: `Controller` → A control loop that watches cluster state and makes/requests changes to move it toward the desired state.  
>
> **Context**: The Redis Enterprise Operator is a custom controller that reconciles Redis Enterprise resources.  
>
> **Example**: Reconciling a `RedisEnterpriseCluster` (REC) after the desired node count changes.  
>
> **Implication**:  
>
> Kubernetes-native reconciliation ensures configuration drift is corrected automatically.

### Core Custom Resources

**RedisEnterpriseCluster (REC):** Declaratively describes a Redis Enterprise cluster on Kubernetes (node count, storage, security, networking, etc.). The operator manages lifecycle (deploy, scale, upgrade, recover). [Redis Enterprise clusters (REC) | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/re-clusters/)

**RedisEnterpriseDatabase (REDB):** Declaratively describes a Redis Enterprise database on an REC (name, memory size, shards, replication, persistence, modules, networking). The operator manages creation, configuration, scaling, and lifecycle. [Redis Enterprise databases (REDB) | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/re-databases/)

**API references:** [RedisEnterpriseCluster API Reference | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/reference/api/redis_enterprise_cluster_api/), [RedisEnterpriseDatabase API Reference | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/reference/api/redis_enterprise_database_api/)

**Namespace scope:** One REC per namespace. [Redis Enterprise for Kubernetes FAQs | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/7.8.4/faqs/)

### Operator Responsibilities

**Deploy:** Creates Kubernetes primitives required for an REC, notably a `StatefulSet` that runs Redis Enterprise nodes (sticky identity and persistent storage). [Operator-based architecture | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/7.4.6/architecture/operator/), [StatefulSets | Kubernetes](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

**Validate:** Enforces constraints such as odd node count and prevents unsafe changes. [Operator-based architecture | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/7.4.6/architecture/operator/), [Size and scale a Redis Enterprise cluster deployment | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/7.8.6/recommendations/sizing-on-kubernetes/)

**Manage & Apply Changes:** Reconciles actual state to match REC/REDB specs (e.g., scaling nodes updates the StatefulSet; changing database shard count triggers resharding by the database controller). [Operator-based architecture | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/7.4.6/architecture/operator/), [Manage Redis Enterprise databases for Kubernetes (db controller) | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/re-databases/db-controller/)

> **Sidenote**  
>
> [Link: Operator-based architecture | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/7.4.6/architecture/operator/)  
>
> **Concept**: `Reconciliation Loop` → The operator continuously reconciles REC/REDB state to match specs.  
>
> **Context**: Any out-of-band changes that conflict with the spec are reverted to the desired state.  
>
> **Example**: If someone scales nodes via the Kubernetes API but not via the REC spec, the operator reconciles back to the REC’s `spec.nodes`.  
>
> **Implication**:  
>
> Treat the CRs as the single source of truth.

### Resources Created by the Operator

- ServiceAccount, Role, RoleBinding (operator and cluster access)
- Secret (REC bootstrap credentials and license)
- StatefulSet (Redis Enterprise nodes)
- Services Manager (aka service rigger) Deployment to expose databases and tag nodes
- Admin UI Service and REST API/Sentinel Service
- Pod Disruption Budget (PDB)
- Optional: Service Broker Deployment and PVC

[Operator-based architecture | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/7.4.6/architecture/operator/), [Manage Redis Enterprise databases for Kubernetes (db controller) | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/re-databases/db-controller/), [Service | Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/), [Configure a Pod Disruption Budget | Kubernetes](https://kubernetes.io/docs/tasks/run-application/configure-pdb/)

### Object Status Lifecycle

**REC status states:** `PendingCreation`, `Initializing`, `Running`, `Error`, `Invalid`.

### Scheduling & High Availability

**StatefulSet semantics:** Stable network identity and storage for each node Pod. [StatefulSets | Kubernetes](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

**Anti-affinity & placement:** Use pod anti-affinity/affinity and spread across zones to avoid single-node failures and improve resiliency. [Assigning Pods to Nodes | Kubernetes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/), [Pod affinity and anti-affinity | Kubernetes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)

**Taints/Tolerations:** Reserve nodes for Redis Enterprise workloads. [Taints and Tolerations | Kubernetes](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

**Rack/Zone awareness:** Keep master/replica shards on separate nodes and consider zone spread. [Shard placement policy | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/memory-performance/shard-placement-policy/)

**Node pools & selectors:** Constrain placement by labels (e.g., high-memory nodes). [Assigning Pods to Nodes | Kubernetes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)

> **Sidenote**  
>
> [Link: Assigning Pods to Nodes | Kubernetes](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)  
>
> **Command**: `nodeSelector` → Constrain Pods to nodes with matching labels.  
>
> **Pattern**: `spec.template.spec.nodeSelector: { "memory": "high" }`  
>
> **Example**: Use a node pool labeled `memory=high` for REC Pods.  
>
> **Result**:  
>
> Redis Enterprise Pods schedule only on nodes with the specified label.

### Persistence

Persistent storage is **mandatory** for Redis Enterprise on Kubernetes. The operator provisions PVCs based on the REC persistence spec and StorageClass. [Redis Enterprise for Kubernetes architecture | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/architecture/), [Persistent Volumes | Kubernetes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

> **Sidenote**  
>
> [Link: Redis Enterprise for Kubernetes architecture | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/architecture/)  
>
> **Concept**: `Persistent storage` → Required for durability and recovery of Redis Enterprise data.  
>
> **Context**: Defined via REC persistence spec and backed by network-attached storage.  
>
> **Example**: Configure `storageClassName` and volume size in the REC spec.  
>
> **Implication**:  
>
> Enables rolling upgrades and disaster recovery.

### Secrets & Credentials

**REC credentials:** Generated and stored in a Kubernetes Secret; name defaults to the REC name. [Manage REC credentials | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/7.8.6/security/manage-rec-credentials/)

**REDB connection info:** The database controller creates a Secret containing `port`, `service_names`, and `password`. The Secret name is recorded in the REDB resource (typically `redb-<db-name>`). [Database connectivity | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/networking/database-connectivity/)

**Kubernetes secrets basics and retrieval:** [Secrets | Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/), [Managing Secrets using kubectl | Kubernetes](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/)

> **Sidenote**  
>
> [Link: Database connectivity | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/networking/database-connectivity/)  
>
> **Command**: `kubectl get redb` → Retrieve database details, including Secret name with credentials.  
>
> **Pattern**: `kubectl get redb <db-name> -o jsonpath="{.spec.databaseSecretName}"`  
>
> **Example**: `kubectl get redb mydb -o jsonpath="{.spec.databaseSecretName}"`  
>
> **Result**:  
>
> Prints the Secret name (e.g., `redb-mydb`) that holds database connection info.

### Networking & Access

**In-cluster access:** The operator creates Services for each database. [Database connectivity | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/networking/database-connectivity/), [Service | Kubernetes](https://kubernetes.io/docs/concepts/services-networking/service/)

**External access:** Configure Ingress (or OpenShift Routes) for TLS/SNI-based routing and external connectivity. [Configure Ingress for external routing | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/networking/ingress/)

> **Sidenote**  
>
> [Link: Configure Ingress for external routing | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/networking/ingress/)  
>
> **Concept**: `Ingress` → Standardized external routing layer for K8s Services (TLS/SNI supported by many controllers).  
>
> **Context**: Route external traffic to REC UI/API and REDB endpoints.  
>
> **Example**: Define host-based rules to route to database Services.  
>
> **Implication**:  
>
> Enables secure, centralized entry points for clients outside the cluster.

### Upgrades

**Process:** Update the Operator, then the REC (cluster), then REDBs (databases). Redis Enterprise uses rolling updates in Kubernetes to minimize downtime. [Upgrade Redis Enterprise for Kubernetes | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/upgrade/), [Performing a Rolling Update | Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)

### Recovery

**Cluster recovery:** Set REC recovery in the spec, ensure Pods reuse original PVCs, then recover databases via `rladmin`. [rladmin recover | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.4/references/cli-utilities/rladmin/recover/)

> **Sidenote**  
>
> [Link: rladmin recover | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.4/references/cli-utilities/rladmin/recover/)  
>
> **Command**: `rladmin recover all` → Recover all databases from persistence.  
>
> **Pattern**: `rladmin recover all`  
>
> **Example**: Inside a Redis Enterprise node Pod: `rladmin recover all`  
>
> **Result**:  
>
> Restores databases from persistence after cluster recovery.

### Active‑Active Databases

Active‑Active (multi‑master) databases replicate across multiple Redis Enterprise clusters for local reads/writes with automatic conflict resolution. Configure the admission controller/webhook and Ingress/Routes. [Active‑Active databases | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/active-active/), [Create Active‑Active database (REAADB) | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/active-active/create-reaadb/), [Establish external routing on the RedisEnterpriseCluster | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/networking/ingressorroutespec/)

> **Sidenote**  
>
> [Link: Create Active‑Active database (REAADB) | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/active-active/create-reaadb/)  
>
> **Concept**: `Admission controller (ValidatingWebhook)` → Validates REDB/REAADB requests before persistence.  
>
> **Context**: Prevents invalid database configurations.  
>
> **Example**: Reject unsupported eviction policies for REDBs.  
>
> **Implication**:  
>
> Guards configuration correctness at the API server layer.

### Supported Kubernetes Distributions

Refer to the current support matrix for Kubernetes distributions and versions. [Supported Kubernetes distributions | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/reference/supported_k8s_distributions/)

### Hands‑On Walkthrough (Key Commands)

```python
# Verify the Kubernetes cluster and nodes are available
# Docs: kubectl get | https://kubernetes.io/docs/reference/kubectl/generated/kubectl_get/
# Docs: kubectl | https://kubernetes.io/docs/reference/kubectl/
print("kubectl get nodes")
print("kubectl version --short")
````

```python
# Install operator dependencies (bundle manifest)
# Docs: kubectl apply | https://kubernetes.io/docs/reference/kubectl/generated/kubectl_apply/
print("kubectl apply -f bundle.yaml")
# Verify operator/controller is running
print("kubectl get pods")
```

```python
# Create the Redis Enterprise Cluster (REC)
# Docs: REC overview | https://redis.io/docs/latest/operate/kubernetes/re-clusters/
print("kubectl apply -f rec.yaml")
# Verify REC pods are coming up
print("kubectl get pods -l app.kubernetes.io/name=redis-enterprise")
```

```python
# Exec into the first REC node pod and check cluster status via rladmin
# Docs: kubectl exec | https://kubernetes.io/docs/reference/kubectl/generated/kubectl_exec/
# Docs: rladmin | https://redis.io/docs/latest/operate/rs/7.8/references/cli-utilities/rladmin/
print("kubectl exec -it rec-0 -- bash -lc 'rladmin status'")
```

```python
# Create a Redis Enterprise Database (REDB)
# Docs: REDB overview | https://redis.io/docs/latest/operate/kubernetes/re-databases/
print("kubectl apply -f redb.yaml")
# Check REDB status and endpoints
# Docs: JSONPath | https://kubernetes.io/docs/reference/kubectl/jsonpath/
print('kubectl get redb mydb -o yaml')
print('kubectl get redb mydb -o jsonpath="{.spec.databaseSecretName}"')
```

```python
# Retrieve REDB password from the Secret and decode (base64)
# Docs: Secrets | https://kubernetes.io/docs/concepts/configuration/secret/
# Docs: Managing Secrets using kubectl | https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/
print("kubectl get secret redb-mydb -o jsonpath='{.data.password}' | base64 --decode; echo")
```

```python
# Port-forward to the database service for local testing
# Docs: kubectl port-forward | https://kubernetes.io/docs/reference/kubectl/generated/kubectl_port-forward/
print("kubectl port-forward service/mydb 16379:19621")
```

```python
# Authenticate and inspect the database via Redis commands
# Docs: AUTH | https://redis.io/docs/latest/commands/auth/
# Docs: INFO | https://redis.io/docs/latest/commands/info/
# r is a connected Redis client; replace with your client initialization.
# concise one-line explanation
r.execute_command("AUTH", "your-db-password")
print("OK")
# concise one-line verification (server info)
r.execute_command("INFO")
print("...INFO output (truncated)...")
```

```python
# Update the REDB spec (e.g., memory size, replication) and apply
# Docs: kubectl apply | https://kubernetes.io/docs/reference/kubectl/generated/kubectl_apply/
print("kubectl apply -f redb.yaml")
# Verify updated status
print("kubectl get redb mydb -o yaml")
```

```python
# Delete the database and then the cluster
# Docs: kubectl delete | https://kubernetes.io/docs/reference/kubectl/generated/kubectl_delete/
print("kubectl delete redb mydb")
print("kubectl delete rec redis-enterprise")
```

### Bibliography

* [Controllers | Kubernetes (Concepts)](https://kubernetes.io/docs/concepts/architecture/controller/)
* [Custom Resources | Kubernetes (Concepts)](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
* [Extend the Kubernetes API with CustomResourceDefinitions | Kubernetes (Tasks)](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/)
* [StatefulSets | Kubernetes (Concepts)](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
* [Redis Enterprise for Kubernetes – Operator-based architecture | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/7.4.6/architecture/operator/)
* [Redis Enterprise clusters (REC) | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/re-clusters/)
* [Redis Enterprise databases (REDB) | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/re-databases/)
* [RedisEnterpriseCluster API Reference | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/reference/api/redis_enterprise_cluster_api/)
* [RedisEnterpriseDatabase API Reference | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/reference/api/redis_enterprise_database_api/)
* [Redis Enterprise for Kubernetes FAQs | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/7.8.4/faqs/)
* [Manage Redis Enterprise databases for Kubernetes (database controller) | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/re-databases/db-controller/)
* [Service | Kubernetes (Concepts)](https://kubernetes.io/docs/concepts/services-networking/service/)
* [Configure a Pod Disruption Budget | Kubernetes (Tasks)](https://kubernetes.io/docs/tasks/run-application/configure-pdb/)
* [Assigning Pods to Nodes (nodeSelector/affinity) | Kubernetes (Concepts)](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/)
* [Pod affinity and anti-affinity | Kubernetes (Concepts)](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)
* [Taints and Tolerations | Kubernetes (Concepts)](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
* [Shard placement policy | Docs - Redis](https://redis.io/docs/latest/operate/rs/databases/memory-performance/shard-placement-policy/)
* [Redis Enterprise for Kubernetes architecture (storage requirement) | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/architecture/)
* [Persistent Volumes | Kubernetes (Concepts)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
* [Secrets | Kubernetes (Concepts)](https://kubernetes.io/docs/concepts/configuration/secret/)
* [Manage Redis Enterprise cluster (REC) credentials | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/7.8.6/security/manage-rec-credentials/)
* [Database connectivity | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/networking/database-connectivity/)
* [Configure Ingress for external routing | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/networking/ingress/)
* [kubectl apply | Kubernetes (Reference)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_apply/)
* [kubectl get | Kubernetes (Reference)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_get/)
* [kubectl exec | Kubernetes (Reference)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_exec/)
* [kubectl port-forward | Kubernetes (Reference)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_port-forward/)
* [JSONPath Support | Kubernetes (Reference)](https://kubernetes.io/docs/reference/kubectl/jsonpath/)
* [Managing Secrets using kubectl | Kubernetes (Tasks)](https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/)
* [AUTH | Docs - Redis](https://redis.io/docs/latest/commands/auth/)
* [INFO | Docs - Redis](https://redis.io/docs/latest/commands/info/)
* [kubectl delete | Kubernetes (Reference)](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_delete/)
* [Upgrade Redis Enterprise for Kubernetes | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/upgrade/)
* [Performing a Rolling Update | Kubernetes (Tutorials)](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)
* [rladmin recover | Docs - Redis](https://redis.io/docs/latest/operate/rs/7.4/references/cli-utilities/rladmin/recover/)
* [Active‑Active databases | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/active-active/)
* [Create Active‑Active database (REAADB) | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/active-active/create-reaadb/)
* [Establish external routing on the RedisEnterpriseCluster | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/networking/ingressorroutespec/)
* [Supported Kubernetes distributions | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/reference/supported_k8s_distributions/)
* [Quick start deployment | Docs - Redis](https://redis.io/docs/latest/operate/kubernetes/deployment/quick-start/)

```
::contentReference[oaicite:0]{index=0}
```
