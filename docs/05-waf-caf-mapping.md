# 05 — Azure Well-Architected Framework & Cloud Adoption Framework Mapping

This is the traceability matrix for CISO, auditor, and procurement conversations: every
architectural decision here is mapped to a specific **WAF pillar** and a specific **CAF
methodology stage**, and to the exact current-state gap it addresses.

## A. Azure Well-Architected Framework — five pillars

### 1. Security

| Current-state gap | Architecture decision | Reference |
|---|---|---|
| Secrets in application config | Azure Key Vault + Managed Identity, zero secrets in code | [02](02-application-architecture.md) §7, [03](03-platform-architecture.md) §3 |
| No SSO; local application accounts | Microsoft Entra ID (External ID + Workload Identity), Conditional Access | [02](02-application-architecture.md) §2, [04](04-networking-security-architecture.md) §5 |
| Public inbound endpoint, no WAF | Azure Front Door Premium + WAF, private endpoints only for APIM/data | [04](04-networking-security-architecture.md) §2, §4 |
| Model/data traffic over public routes | Hub-spoke VNets, private endpoints, Firewall FQDN-restricted egress | [04](04-networking-security-architecture.md) §1, §3, §4 |
| No vulnerability/posture scanning | Defender for Cloud (CSPM + workload plans), ACR image scanning | [04](04-networking-security-architecture.md) §6 |
| No documented threat model | RAG/agent threat-model notes + guardrails + firewall allow-list | [04](04-networking-security-architecture.md) §7 |

### 2. Reliability

| Current-state gap | Architecture decision | Reference |
|---|---|---|
| Single VM, single point of failure | Azure Container Apps, zone-redundant, autoscaled | [02](02-application-architecture.md) §3 |
| No backup/restore test | Automated backup for PostgreSQL/Blob with tested restore, documented RTO/RPO | [06](06-migration-roadmap.md) phase 3 |
| No DR plan | Deployment Stamps pattern enables a secondary-region stamp; DR runbook | [01](01-multitenancy-strategy.md) §2, [06](06-migration-roadmap.md) phase 4 |
| No load/soak testing | Load & chaos testing gated into CI/CD before prod promotion | [03](03-platform-architecture.md) §4 |

### 3. Cost Optimization

| Current-state gap | Architecture decision | Reference |
|---|---|---|
| Cost rises faster than capacity (vertical-only scaling) | Elastic ACA autoscaling — pay for consumption, not a fixed bigger VM | [02](02-application-architecture.md) §3 |
| No cost visibility per tenant | Resource tagging (`tenant_id`,`tier`,`env`) + Cost Management + chargeback | [03](03-platform-architecture.md) §2, [01](01-multitenancy-strategy.md) §3 |
| No provisioned-vs-consumption model | Explicit open design item: PTU vs consumption pricing for Azure OpenAI | [06](06-migration-roadmap.md) open items |

### 4. Operational Excellence

| Current-state gap | Architecture decision | Reference |
|---|---|---|
| No IaC | Bicep/Terraform per landing zone/stamp, pipeline-only deploys | [03](03-platform-architecture.md) §3 |
| No environment separation | Dev/Test/Staging/Prod, each in its own landing-zone subscription | [03](03-platform-architecture.md) §5 |
| No formal change process | Mandatory pipeline approvals for prod; automated validation gates | [03](03-platform-architecture.md) §4, §5 |
| No centralized detection | Microsoft Sentinel SIEM/SOAR across Front Door/Firewall/Entra/APIM logs | [04](04-networking-security-architecture.md) §6 |

### 5. Performance Efficiency

| Current-state gap | Architecture decision | Reference |
|---|---|---|
| Model throughput limits unknown | Capacity model workstream (PTU sizing) before GA | [06](06-migration-roadmap.md) open items |
| No autoscaling or queueing | KEDA-scaled ACA + Service Bus for async workers; APIM rate limiting for graceful degradation | [02](02-application-architecture.md) §3, §4 |

## B. Cloud Adoption Framework — methodology mapping

| CAF stage | What this repository defines |
|---|---|
| **Strategy** | Business justification implicit in the multi-tenancy strategy (pool vs. silo as a commercial tier) — [01](01-multitenancy-strategy.md) |
| **Plan** | Migration roadmap with phased workstreams and owners — [06](06-migration-roadmap.md) |
| **Ready** (landing zones) | Management group hierarchy, subscription design, hub-spoke network — [03](03-platform-architecture.md) §1, [04](04-networking-security-architecture.md) §1 |
| **Migrate/Innovate** | Application re-platforming from VM/Docker Compose to Azure Container Apps — [02](02-application-architecture.md) |
| **Govern** | Azure Policy, tagging standard, RBAC + PIM, Microsoft Purview data governance — [03](03-platform-architecture.md) §3 |
| **Manage** | Azure Monitor/Log Analytics, Defender for Cloud, Sentinel, FinOps (Cost Management) — [03](03-platform-architecture.md) §2, [04](04-networking-security-architecture.md) §6 |
