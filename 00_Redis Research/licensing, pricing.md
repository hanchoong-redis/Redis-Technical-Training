# Redis Licensing, Pricing, and Delivery Models — Sales Research Report (2025)

---

## 1) Executive Summary
- **Licensing:** Redis **8.0+** (“Redis Open Source”) is **tri-licensed**: **RSALv2**, **SSPLv1**, **AGPLv3**. Redis **≤7.2** remained **BSD‑3‑Clause**; **7.4–7.8** (“Community Edition”) used **RSALv2/SSPLv1** dual-license. Starting with Redis 8, core modules (RediSearch, RedisJSON, RedisTimeSeries, RedisBloom) are integrated and follow the same tri-license. **Redis proprietary products** (Redis Cloud, Redis Software) require commercial licenses. [Licenses | Redis](https://redis.io/legal/licenses/)  
- **Pricing:**  
  - **Redis Enterprise Cloud (SaaS)**: Free ($0, 30 MB), **Flex** (from **$0.007/hr ≈ $5/mo**), **Essentials** (from **$0.007/hr ≈ $5/mo**), **Pro** (from **$0.274/hr ≈ $200/mo**). **Pro** is **dedicated**, includes **Active‑Active**, auto‑tiering, private connectivity, and **99.999% uptime SLA**; Essentials/Flex are shared with **99.99% uptime**. [Let’s talk numbers | Redis](https://redis.io/pricing/); [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/)  
  - **Redis Enterprise Software (self‑managed)**: **Annual subscriptions priced by shard count** (masters, replicas, and cluster shards all count). **Custom quote required**. **30‑day free trial** available for evaluation. [Software Pricing | Redis](https://redis.io/enterprise/pricing/)  
- **Delivery models:**  
  - **Self‑managed** (VMs/Kubernetes via **Redis Enterprise Operator**). [Redis Enterprise for Kubernetes | Docs](https://redis.io/docs/latest/operate/kubernetes/)  
  - **Fully managed SaaS** (**Redis Cloud**) on **AWS/Azure/GCP**; also **BYOC/Private Cloud** option via **Cloud Agreement Appendix 1**. [Redis Cloud](https://redis.io/cloud/); [Redis Cloud Agreement](https://redis.io/legal/cloud-tos/)  
  - **Marketplaces:** AWS PAYG with **14‑day free trial** + **Annual private offers**; GCP/Azure marketplace procurement and mapping. [AWS Marketplace – Redis Cloud PAYG](https://aws.amazon.com/marketplace/pp/prodview-mwscixe4ujhkq); [AWS Marketplace – Annual by Redis](https://aws.amazon.com/marketplace/pp/prodview-k6njijz4vkdac); [GCP Marketplace mapping (docs)](https://redis.io/docs/latest/operate/rc/cloud-integrations/gcp-marketplace/); [Azure Managed Redis](https://azure.microsoft.com/en-us/products/managed-redis/)  
- **Sales angles:** **Enterprise SLAs (up to 99.999%)**, **Active‑Active/CRDT geo‑distribution**, **auto‑tiering (DRAM+SSD)**, **private connectivity**, and **compliance** (SOC 2, HIPAA, PCI, ISO 27001/17/18, GDPR) differentiate against do‑it‑yourself OSS and hyperscaler look‑alikes. [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/); [Active‑Active | Docs](https://redis.io/docs/latest/operate/rs/databases/active-active/); [Auto Tiering | Docs](https://redis.io/docs/latest/operate/rs/databases/auto-tiering/); [Trust Center](https://trust.redis.io/)  
- **Ecosystem context:** Redis’ 2024 relicense (RSAL/SSPL) led to the **Valkey** fork (BSD‑3 under Linux Foundation) adopted by AWS/Google; in 2025 Redis added **AGPLv3** in a tri‑license for **Redis 8.0+**. [Licenses | Redis](https://redis.io/legal/licenses/); [Linux Foundation Valkey announcement](https://www.linuxfoundation.org/press/linux-foundation-launches-open-source-valkey-community); [AWS Database Blog – Valkey](https://aws.amazon.com/blogs/database/year-one-of-valkey-open-source-innovations-and-elasticache-version-8-1-for-valkey/); [Google Cloud – Memorystore for Valkey GA](https://cloud.google.com/blog/products/databases/announcing-general-availability-of-memorystore-for-valkey)

---

## 2) Methodology & Sources Used
This report draws primarily from **Redis legal documentation**, **Redis pricing pages**, **Redis product docs**, and **cloud marketplace listings** (AWS, Azure, GCP), supplemented by credible vendor blogs/press. All claims are cited inline. [Licenses | Redis](https://redis.io/legal/licenses/); [Let’s talk numbers | Redis](https://redis.io/pricing/); [Software Pricing | Redis](https://redis.io/enterprise/pricing/); [Redis Cloud Agreement](https://redis.io/legal/cloud-tos/); [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/); [AWS Marketplace PAYG](https://aws.amazon.com/marketplace/pp/prodview-mwscixe4ujhkq); [Azure Managed Redis](https://azure.microsoft.com/en-us/products/managed-redis/); [GCP Marketplace mapping](https://redis.io/docs/latest/operate/rc/cloud-integrations/gcp-marketplace/)

---

## 3) Licensing Models

### 3.1 Overview & Evolution
- **Tri-license from Redis 8.0+**: Users may choose **RSALv2**, **SSPLv1**, or **AGPLv3** for Redis Open Source 8+, enabling both source‑available and OSI‑approved OSS paths. [Licenses | Redis](https://redis.io/legal/licenses/)  
- **Prior versions**: **≤7.2** were **BSD‑3**; **7.4–7.8** were **RSALv2/SSPLv1** dual‑licensed. [Licenses | Redis](https://redis.io/legal/licenses/)  
- **Modules**: RediSearch, RedisJSON, RedisTimeSeries, RedisBloom became **integral to Redis Open Source** with Redis 8 and follow the same tri‑license; the legal page provides detailed historical module licensing (AGPL, Commons Clause, RSAL v1/v2, SSPL). [Licenses | Redis](https://redis.io/legal/licenses/)

### 3.2 RSALv2 (Redis Source Available License v2)
- **Permissive source-available** with two key restrictions: you may not **commercialize or offer it as a managed service** providing the software’s functionality to third parties, and you must **retain notices**. **Not an OSI open-source license**. [Licenses | Redis](https://redis.io/legal/licenses/)

### 3.3 SSPLv1 (Server Side Public License)
- **Copyleft** (based on GPLv3) requiring that if you **offer the product as a service**, you must release **modifications and management layers** under SSPL. **Not an OSI open-source license**. [Licenses | Redis](https://redis.io/legal/licenses/); [MongoDB SSPL FAQ](https://www.mongodb.com/legal/licensing/server-side-public-license/faq)

### 3.4 AGPLv3
- **OSI‑approved copyleft** designed for network software; requires making source (including modifications) available to **users interacting over a network**. [Licenses | Redis](https://redis.io/legal/licenses/); [AGPLv3 (FSF)](https://www.gnu.org/licenses/agpl-3.0.en.html)

### 3.5 Modules & Tools Licensing
- **Modules** (e.g., RediSearch/RedisJSON/RedisTimeSeries/RedisBloom) are tri‑licensed with Redis 8; legal history shows earlier versions under AGPL, Apache+Commons Clause, RSAL v1/v2, SSPL. [Licenses | Redis](https://redis.io/legal/licenses/)  
- **Tools**: Redis Insight and Redis for VS Code are **commercial for paying customers**; **community users** use them under **SSPLv1**. [Licenses | Redis](https://redis.io/legal/licenses/)

### 3.6 Impact on Cloud Vendors & Partners
- **Valkey (Linux Foundation)** forked from Redis 7.2.4 under **BSD‑3** following the 2024 Redis license shift; major clouds (AWS/Google) adopted Valkey‑based managed services. [Linux Foundation Valkey announcement](https://www.linuxfoundation.org/press/linux-foundation-launches-open-source-valkey-community); [AWS Database Blog – Valkey](https://aws.amazon.com/blogs/database/year-one-of-valkey-open-source-innovations-and-elasticache-version-8-1-for-valkey/); [Google Cloud – Memorystore for Valkey GA](https://cloud.google.com/blog/products/databases/announcing-general-availability-of-memorystore-for-valkey)

### 3.7 Redis Enterprise Proprietary Licenses
- **Redis Software (self‑managed)** and **Redis Cloud (SaaS)** are **closed-source commercial offerings** under their respective agreements; Redis notes its proprietary products require commercial licenses. [Licenses | Redis](https://redis.io/legal/licenses/)  
- **Software Agreement (EULA)** governs self‑managed use; license is non‑transferable, non‑sublicensable, with stated restrictions and support terms. [Software Agreement | Redis](https://redis.io/legal/software-agreement/)  
- **Cloud Agreement (ToS)** governs Redis Cloud access (definitions, fees, committed expenditure, SLA linkage, BYOC Appendix). [Redis Cloud Agreement](https://redis.io/legal/cloud-tos/)

---

## 4) Pricing Models

### 4.1 Redis Enterprise Software (self‑managed)
- **How it’s priced:** Annual subscriptions **by shard**; a shard includes **masters, replicas, or cluster partitions**—“any Redis process provisioned by you with Redis Enterprise.” **Price per shard** varies by product, shard type, prod vs non‑prod, and total shards. **Custom quote required**. [Software Pricing | Redis](https://redis.io/enterprise/pricing/)  
- **Trial:** **30‑day free trial** for evaluation/dev (not for production/QA). [Software Pricing | Redis](https://redis.io/enterprise/pricing/)

### 4.2 Redis Enterprise Cloud (SaaS)
- **Plans & starting prices** (on‑demand unless annual):  
  - **Free**: $0/hr; **30 MB** single DB; best‑effort SLA. [Let’s talk numbers | Redis](https://redis.io/pricing/)  
  - **Flex**: from **$0.007/hr (~$5/mo)**, shared; 1–100 GB (RAM+SSD); SAML SSO/RBAC; **99.99% uptime**, basic support. [Let’s talk numbers | Redis](https://redis.io/pricing/)  
  - **Essentials**: from **$0.007/hr (~$5/mo)**, shared; **250 MB–12 GB RAM**, single DB; SAML SSO/RBAC; **99.99% uptime**, basic support. [Let’s talk numbers | Redis](https://redis.io/pricing/)  
  - **Pro**: from **$0.274/hr (~$200/mo)**, **dedicated**; **multi‑DB**, **Active‑Active**, **auto‑tiering**, **private connectivity**, **99.999% uptime**, 24/7 hotline. [Let’s talk numbers | Redis](https://redis.io/pricing/); [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/)  
- **SLA tiers:** **99.999%** (Active‑Active), **99.99%** (Multi‑AZ), **99.9%** (Standard); credits are the sole remedy. [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/)

### 4.3 Marketplaces & Trials
- **AWS Marketplace**: **Pay‑as‑you‑go with 14‑day free trial**; **Annual** private‑offer SKU for committed spend. [AWS Marketplace – Redis Cloud PAYG](https://aws.amazon.com/marketplace/pp/prodview-mwscixe4ujhkq); [AWS Marketplace – Annual by Redis](https://aws.amazon.com/marketplace/pp/prodview-k6njijz4vkdac)  
- **GCP Marketplace**: Subscribe/m**ap billing** to GCP account (consolidated invoice, private offers). [GCP Marketplace mapping (docs)](https://redis.io/docs/latest/operate/rc/cloud-integrations/gcp-marketplace/)  
- **Azure**: **Azure Managed Redis** (first‑party service based on Redis Enterprise) procured via Azure; also historic Redis Enterprise marketplace SKUs. [Azure Managed Redis](https://azure.microsoft.com/en-us/products/managed-redis/); [Azure Marketplace — Redis Enterprise listing](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/garantiadata.redisvpc?tab=Overview)

### 4.4 How Scaling Impacts Price
- **Shards are the billing unit** in both Cloud (defined in SLA) and Software (EULA/pricing); **replicas** and **cluster partitioning** increase shard count → higher cost. [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/); [Software Pricing | Redis](https://redis.io/enterprise/pricing/)  
- **Data transfer**: On on‑demand plans, **limited data transfer is included**; annual/enterprise may have **usage‑based** networking (see pricing table). [Let’s talk numbers | Redis](https://redis.io/pricing/)  
- **Workload fit & upgrades**: SLA notes **Provisioned Workload** vs **Actual Workload**; Redis may require plan/shard increases after a grace period to meet demand. [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/)

### 4.5 Custom Enterprise Pricing & Commitments
- **Commercial Software**: **Annual** shard‑based subscriptions via direct or marketplace with support tiers. **Quote required**. [Software Pricing | Redis](https://redis.io/enterprise/pricing/)  
- **Cloud**: **Committed Expenditure** and **usage** components; private offers via marketplaces/partners; pass‑through network charges in certain cases; BYOC terms in **Appendix 1**. [Redis Cloud Agreement](https://redis.io/legal/cloud-tos/)

---

## 5) Delivery Models

### 5.1 Self‑Managed (On‑Prem/Kubernetes)
- **Redis Enterprise Software** deploys on VMs or **Kubernetes** using the **Redis Enterprise Operator** (CRDs, Helm); supports persistence, security, and scale‑out clustering. [Redis Enterprise for Kubernetes | Docs](https://redis.io/docs/latest/operate/kubernetes/); [Kubernetes Operator Architecture](https://redis.io/docs/latest/operate/kubernetes/architecture/); [Helm install](https://redis.io/docs/latest/operate/kubernetes/deployment/helm/)  

### 5.2 Fully Managed (Redis Enterprise Cloud SaaS)
- **Multi‑cloud** (AWS/Azure/GCP), **Pro** is **dedicated**; features include **Active‑Active**, auto‑tiering, private connectivity (e.g., **AWS PrivateLink/VPC endpoints**), and enterprise SLAs. [Let’s talk numbers | Redis](https://redis.io/pricing/); [Redis Cloud](https://redis.io/cloud/); [AWS VPC endpoints for Redis Cloud](https://redis.io/blog/aws-vpc-endpoints-now-available-on-redis-cloud/)

### 5.3 Hybrid / BYOC
- Redis Cloud supports **Bring Your Own Cloud (Private Cloud Deployment)**: Redis runs inside the customer’s cloud account with additional terms in **Appendix 1**; multi‑cloud & hybrid deployment options are available with annual plans. [Redis Cloud Agreement](https://redis.io/legal/cloud-tos/); [Let’s talk numbers | Redis](https://redis.io/pricing/)

### 5.4 Marketplace Availability
- **AWS**: Redis Cloud **Pay‑as‑you‑go** & **Annual** private‑offer SKUs. [AWS Marketplace – Redis Cloud PAYG](https://aws.amazon.com/marketplace/pp/prodview-mwscixe4ujhkq); [AWS Marketplace – Annual by Redis](https://aws.amazon.com/marketplace/pp/prodview-k6njijz4vkdac)  
- **Azure**: **Azure Managed Redis** by Microsoft (based on Redis Enterprise), with global availability and Azure‑native ops/billing. [Azure Managed Redis](https://azure.microsoft.com/en-us/products/managed-redis/); [Quickstart (docs)](https://learn.microsoft.com/en-us/azure/redis/quickstart-create-managed-redis)  
- **GCP**: **GCP Marketplace** subscription with billing **mapping** to Redis Cloud accounts. [GCP Marketplace mapping (docs)](https://redis.io/docs/latest/operate/rc/cloud-integrations/gcp-marketplace/)

---

## 6) Sales Enablement

### 6.1 Positioning: Redis Open Source vs Redis Enterprise
- **Enterprise differentiation**: **Active‑Active/CRDT** multi‑region writes, **auto‑tiering** (DRAM+SSD), dedicated **Pro** deployments, **private connectivity**, and **SLA up to 99.999%** are Enterprise/Cloud value props **not present** in Redis OSS alone. [Active‑Active | Docs](https://redis.io/docs/latest/operate/rs/databases/active-active/); [Auto Tiering | Docs](https://redis.io/docs/latest/operate/rs/databases/auto-tiering/); [Let’s talk numbers | Redis](https://redis.io/pricing/); [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/)  

### 6.2 Common Objections (What to address)
- **“Why not just OSS Redis?”** → OSS lacks **enterprise SLAs**, **managed operations**, **CRDT Active‑Active**, and **auto‑tiering**; Redis Cloud/Software provide these plus **24/7 support** and **compliance**. [Active‑Active | Docs](https://redis.io/docs/latest/operate/rs/databases/active-active/); [Auto Tiering | Docs](https://redis.io/docs/latest/operate/rs/databases/auto-tiering/); [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/); [Trust Center](https://trust.redis.io/)  
- **“Is compliance covered?”** → Redis publishes certifications and attestations (e.g., **SOC 2**, **HIPAA**, **PCI DSS**, **ISO/IEC 27001/17/18**, **GDPR**) in its **Trust Center**. [Trust Center](https://trust.redis.io/)  
- **“TCO vs DIY?”** → AWS/Redis migration guidance highlights the **operational overhead** of self‑managed Redis and recommends managed approaches for scale and reliability. [AWS Prescriptive Guidance – Migrate to Redis Cloud](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/migrate-redis-workloads-to-redis-enterprise-cloud-on-aws.html)

### 6.3 Compliance & Certifications
- Redis lists certifications and compliance frameworks (SOC 2, HIPAA, PCI, ISO/IEC 27001/17/18, GDPR, etc.) in its **Trust Center**. [Trust Center](https://trust.redis.io/)

### 6.4 TCO/ROI Talking Points
- **Managed service** reduces **operational complexity** (maintenance, patching, scaling), offers **predictable SLAs**, and frees teams to focus on app features; these benefits are emphasized in AWS prescriptive guidance for Redis Cloud migrations. [AWS Prescriptive Guidance – Migrate to Redis Cloud](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/migrate-redis-workloads-to-redis-enterprise-cloud-on-aws.html)

---

## 7) Comparative Analysis

### 7.1 Licensing Comparison (Redis vs Others)
- **Redis 8+**: **Tri‑licensed** (RSALv2/SSPLv1/**AGPLv3**), re‑introducing an **OSI‑approved** option (**AGPLv3**) alongside source‑available options. [Licenses | Redis](https://redis.io/legal/licenses/)  
- **MongoDB**: **SSPLv1** (non‑OSI), requiring open‑sourcing of management layers when offered “as‑a‑service.” [MongoDB SSPL FAQ](https://www.mongodb.com/legal/licensing/server-side-public-license/faq)  
- **Elastic**: **Elastic License 2.0** and (historically) **SSPL** in certain dual‑license contexts; ELv2 disallows offering as a **managed service**. [Elastic License FAQ](https://www.elastic.co/licensing/elastic-license/faq)

### 7.2 SaaS Delivery Differentiation
- Redis Cloud **Pro** emphasizes **dedicated** deployments, **Active‑Active** geo‑replication, **auto‑tiering**, and **private connectivity** with **99.999%** SLA; hyperscaler-native services often focus on Redis‑compatible engines (including **Valkey**) and hyperscaler‑specific integration/billing. [Let’s talk numbers | Redis](https://redis.io/pricing/); [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/); [AWS Database Blog – Valkey](https://aws.amazon.com/blogs/database/year-one-of-valkey-open-source-innovations-and-elasticache-version-8-1-for-valkey/); [Google – Memorystore for Valkey GA](https://cloud.google.com/blog/products/databases/announcing-general-availability-of-memorystore-for-valkey)

---

## 8) Customer‑Fit Guidance

| Customer profile | Recommended Redis model | Why it fits (key capabilities) |
|---|---|---|
| **Startup / SMB** building first cache or feature flags | **Redis Cloud Free/Flex** | **Low cost** entry (Free **30 MB**; Flex from **$5/mo**), managed operations, **99.99%** uptime, SSO/RBAC. [Let’s talk numbers | Redis](https://redis.io/pricing/) |
| **Mid‑market** app teams needing single DB at modest size | **Redis Cloud Essentials** | Shared plan with **250 MB–12 GB RAM** single DB, **99.99%** uptime, security features. [Let’s talk numbers | Redis](https://redis.io/pricing/) |
| **Enterprise** with multi‑region RTO/RPO targets | **Redis Cloud Pro** | **Dedicated** clusters, **Active‑Active/CRDT**, **auto‑tiering**, **private connectivity**, **99.999%** uptime SLA, 24/7 hotline. [Let’s talk numbers | Redis](https://redis.io/pricing/); [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/); [Active‑Active | Docs](https://redis.io/docs/latest/operate/rs/databases/active-active/) |
| **Regulated / data‑residency** or tight network control | **Redis Cloud BYOC** or **Redis Software** | BYOC runs **inside your cloud account**; self‑managed offers full control on‑prem/Kubernetes with enterprise support. [Redis Cloud Agreement](https://redis.io/legal/cloud-tos/); [Redis Enterprise for Kubernetes | Docs](https://redis.io/docs/latest/operate/kubernetes/) |
| **Hyperscaler commit drawdown** procurement | **Marketplace SKUs** | AWS **PAYG (14‑day trial)**, **Annual private offers**; GCP mapping; Azure first‑party **Managed Redis**. [AWS PAYG](https://aws.amazon.com/marketplace/pp/prodview-mwscixe4ujhkq); [AWS Annual](https://aws.amazon.com/marketplace/pp/prodview-k6njijz4vkdac); [GCP mapping](https://redis.io/docs/latest/operate/rc/cloud-integrations/gcp-marketplace/); [Azure Managed Redis](https://azure.microsoft.com/en-us/products/managed-redis/) |

---

## 9) Appendices

### A) License Evolution Timeline (Condensed)
- **≤7.2** — **BSD‑3‑Clause** (“Redis” classic OSS). [Licenses | Redis](https://redis.io/legal/licenses/)  
- **7.4–7.8** — **RSALv2/SSPLv1** dual‑licensed (“Redis Community Edition”). [Licenses | Redis](https://redis.io/legal/licenses/)  
- **8.0+** — **Tri‑license** (**RSALv2**, **SSPLv1**, **AGPLv3**); modules (RediSearch/RedisJSON/RedisTimeSeries/RedisBloom) integrated and tri‑licensed. [Licenses | Redis](https://redis.io/legal/licenses/)  
- **Ecosystem** — **Valkey** launches under **BSD‑3** at Linux Foundation (fork of 7.2.4) after 2024 changes; adopted by AWS/Google managed services. [Linux Foundation Valkey announcement](https://www.linuxfoundation.org/press/linux-foundation-launches-open-source-valkey-community); [AWS Database Blog – Valkey](https://aws.amazon.com/blogs/database/year-one-of-valkey-open-source-innovations-and-elasticache-version-8-1-for-valkey/); [Google – Memorystore for Valkey GA](https://cloud.google.com/blog/products/databases/announcing-general-availability-of-memorystore-for-valkey)

### B) Delivery Models — Side‑by‑Side

| Model | Who runs it | Where it runs | Payment | SLA | Notes |
|---|---|---|---|---|---|
| **Redis Software (self‑managed)** | Customer | On‑prem/VMs/Kubernetes (Operator/Helm) | **Annual shard‑based** subscription (**quote required**) | N/A (customer‑operated) | EULA governs; shards include masters/replicas/partitions. [Software Pricing | Redis](https://redis.io/enterprise/pricing/); [Software Agreement](https://redis.io/legal/software-agreement/); [K8s Operator Docs](https://redis.io/docs/latest/operate/kubernetes/) |
| **Redis Cloud – Free** | Redis | AWS/Azure/GCP | $0 (30 MB) | Best‑effort | Single DB, community support. [Let’s talk numbers | Redis](https://redis.io/pricing/) |
| **Redis Cloud – Flex** | Redis | Shared infra | From **$0.007/hr (~$5/mo)** | **99.99%** | 1–100 GB (RAM+SSD), SSO/RBAC. [Let’s talk numbers | Redis](https://redis.io/pricing/); [SLA](https://redis.io/legal/redis-cloud-service-level-agreement/) |
| **Redis Cloud – Essentials** | Redis | Shared infra | From **$0.007/hr (~$5/mo)** | **99.99%** | 250 MB–12 GB RAM, single DB. [Let’s talk numbers | Redis](https://redis.io/pricing/) |
| **Redis Cloud – Pro** | Redis | **Dedicated** | From **$0.274/hr (~$200/mo)** | **Up to 99.999%** | **Active‑Active**, auto‑tiering, private connectivity. [Let’s talk numbers | Redis](https://redis.io/pricing/); [SLA](https://redis.io/legal/redis-cloud-service-level-agreement/) |
| **Redis Cloud – BYOC** | Redis (in your VPC) | Customer’s cloud account | Contractual commit + usage | As per plan | **Private Cloud Deployment** terms (Appendix 1). [Cloud Agreement](https://redis.io/legal/cloud-tos/) |
| **Marketplace (AWS/GCP/Azure)** | Redis/Microsoft | Hyperscaler portals | PAYG, annual, private offers | As per plan | Billing mapped to cloud commits; Azure offers **Managed Redis** (first‑party). [AWS PAYG](https://aws.amazon.com/marketplace/pp/prodview-mwscixe4ujhkq); [AWS Annual](https://aws.amazon.com/marketplace/pp/prodview-k6njijz4vkdac); [GCP mapping](https://redis.io/docs/latest/operate/rc/cloud-integrations/gcp-marketplace/); [Azure Managed Redis](https://azure.microsoft.com/en-us/products/managed-redis/)

### C) Quick Objection‑Handling FAQ

- **Q: “Can we repackage Redis 8 as our managed service?”**  
  **A:** Under **RSALv2**, you may not commercialize or provide it “as‑a‑service” to third parties; **SSPLv1** requires releasing your management layers’ source if offered as a service; **AGPLv3** is open source but carries copyleft obligations. Choose the license that fits your use; Redis proprietary products require commercial licensing. [Licenses | Redis](https://redis.io/legal/licenses/)

- **Q: “What uptime can we guarantee?”**  
  **A:** Redis Cloud **SLA** targets **99.999%** for **Active‑Active**, **99.99%** for **Multi‑AZ**, and **99.9%** for **Standard** deployments, with **service credits** as the remedy. [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/)

- **Q: “How does pricing scale with replicas and clusters?”**  
  **A:** **Shards** are the billing unit (Cloud & Software), and **replicas/partitions** count as additional shards; more shards → higher cost. [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/); [Software Pricing | Redis](https://redis.io/enterprise/pricing/)

- **Q: “What compliance assurances can we share?”**  
  **A:** Redis publishes **SOC 2**, **HIPAA**, **PCI DSS**, **ISO/IEC 27001/17/18**, **GDPR** and more in the **Trust Center**. [Trust Center](https://trust.redis.io/)

---

## 10) Reference Links (select highlights)
- **Licensing hub:** [Licenses | Redis](https://redis.io/legal/licenses/)  
- **Pricing (Cloud):** [Let’s talk numbers | Redis](https://redis.io/pricing/)  
- **Pricing (Software):** [Software Pricing | Redis](https://redis.io/enterprise/pricing/)  
- **Cloud terms & SLA:** [Redis Cloud Agreement](https://redis.io/legal/cloud-tos/); [Redis Cloud SLA](https://redis.io/legal/redis-cloud-service-level-agreement/)  
- **Enterprise features:** [Active‑Active | Docs](https://redis.io/docs/latest/operate/rs/databases/active-active/); [Auto Tiering | Docs](https://redis.io/docs/latest/operate/rs/databases/auto-tiering/)  
- **Operator/Kubernetes:** [Redis Enterprise for Kubernetes](https://redis.io/docs/latest/operate/kubernetes/)  
- **Compliance:** [Trust Center](https://trust.redis.io/)  
- **Marketplaces:** [AWS Marketplace – PAYG](https://aws.amazon.com/marketplace/pp/prodview-mwscixe4ujhkq); [AWS – Annual](https://aws.amazon.com/marketplace/pp/prodview-k6njijz4vkdac); [GCP mapping](https://redis.io/docs/latest/operate/rc/cloud-integrations/gcp-marketplace/); [Azure Managed Redis](https://azure.microsoft.com/en-us/products/managed-redis/)  

---
