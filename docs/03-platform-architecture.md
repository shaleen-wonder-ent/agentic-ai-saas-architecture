# 03 — Platform Architecture

**Diagram:** [`diagrams/02-platform-architecture.drawio`](../diagrams/02-platform-architecture.drawio)

## Purpose

Defines the **Cloud Adoption Framework (CAF) landing zone** that hosts the platform, and
the **SaaS control plane** (platform services that operate the product day to day) —
modelled directly on the customer's own internal **"AI Asset Fabric"** reference
architecture so the target architecture speaks the same language the business already
uses internally.

This is the direct answer to the readiness pack's Governance pillar gap: *"Single
subscription, no landing zone... no Azure Policy or tagging standard... broad standing
access... no formal change process... residency undocumented."*

## 1. Management group and subscription design (CAF landing zone)

Following the [CAF landing zone reference](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/landing-zone/):

```
Tenant Root Management Group
├── Platform Management Group
│   ├── Connectivity Subscription   — hub VNet, Azure Firewall, DNS, VPN/ExpressRoute
│   ├── Identity Subscription       — Microsoft Entra ID config, PIM, Conditional Access
│   └── Management Subscription     — Log Analytics, Azure Automation, Azure Policy assignments
├── Landing Zones Management Group
│   ├── Production Subscription       — prod app spoke + data spoke
│   └── Non-Production Subscription   — dev / test / staging
└── Sandbox Management Group                     — isolated experimentation, no path to prod data
```

Rationale: a **single-product estate** (per the customer's own open design question) still
benefits from separating Connectivity/Identity/Management from the product's own
subscriptions, because it (a) lets platform-wide policy and network controls be managed
once, (b) keeps blast radius of a compromised prod subscription away from identity/hub
resources, and (c) gives clean cost attribution per environment from day one.

## 2. SaaS control plane — "AI Asset Fabric" for Azure

The customer's own internal platform architecture already defines the control-plane
functions the product needs. This architecture maps each function to concrete Azure
services so it can be built, not just diagrammed:

| AI Asset Fabric function | Azure realization |
|---|---|
| **Governance & Control** — Control Tower, Guardrails, AI Governance, Audit & Evidence | Azure Workbooks (ops dashboard) + the existing guardrail/audit-ledger services (now ACA-hosted) + Microsoft Purview for compliance evidence |
| **Agent Lifecycle** — Agent Builder/Playgrounds, Testing Lab, Use-Case Marketplace | Internal low-code studio + connector test harness, deployed as an ACA app; use-case registry backing the "new workflows without platform redeploy" pattern |
| **Connectivity** — MCP Gateway, Tool Connectors, API Gateway, Event Framework | MCP tool layer (ACA) + **Azure API Management** (API/AI Gateway) + **Azure Event Grid** for event-driven triggers |
| **Administration** — Admin Config, Tenant Management, Tenants & API Keys | Tenant Management Service (Azure SQL/Cosmos DB-backed registry) + Azure App Configuration + Key Vault/APIM subscriptions for per-tenant isolation |
| **Observability & Governance** — Observability, AI Governance, Audit & Evidence | Azure Monitor + Log Analytics (logs/traces/metrics) + Microsoft Purview |
| **FinOps** — Cost & usage control, Budgeting & Alerts, Chargeback | Azure Cost Management + Budgets, resource tagging (`tenant_id`, `tier`, `env`) driving chargeback/showback reports |

## 3. Foundational platform services (CAF "platform automation and DevOps")

- **Azure Policy** — enforces the tagging standard, denies public IPs on PaaS, enforces
  approved SKUs/regions, and is the mechanism (not manual review) that keeps every
  subscription compliant with the baseline agreed with the customer's security team.
- **Microsoft Entra ID** — identity foundation shared by workforce (the platform's
  build/ops team, least-privilege RBAC with just-in-time elevation via PIM) and workload
  identities.
- **Azure Key Vault** — one per environment (pool) / per silo tenant, secrets accessed
  only via Managed Identity.
- **Microsoft Defender for Cloud** — subscription-wide posture management plus
  workload-specific plans (Defender for APIs, Defender for Containers, Defender for Key
  Vault, Defender for DNS).
- **Azure Container Registry** — the only path container images take into the platform,
  scanned for vulnerabilities before deployment.
- **IaC + CI/CD** — Bicep or Terraform modules per landing zone/stamp, deployed only via
  Azure DevOps or GitHub Actions pipelines with mandatory approvals. This is the
  **only change path** — replacing the MVP's "manual deployment, no rollback path."

## 4. Platform lifecycle

The platform operates the six-stage lifecycle already presented to customers in the
vendor's own rollout journey — **Select Use Case → Configure → Integrate → Validate →
Deploy → Scale** — and this repository's platform architecture is what makes each stage
repeatable and auditable rather than a manual, bespoke engagement per customer:

1. **Select Use Case** — tenant onboarding selects from the use-case marketplace
   (Claims Triaging, Submission Center, Document Intelligence, etc.).
2. **Configure** — Admin Config applies tenant-specific guardrails, quotas, and
   integration endpoints.
3. **Integrate** — MCP Gateway/Tool Connectors wire up the tenant's CRM/ERP/email systems.
4. **Validate** — automated functional, performance, and security validation gates in the
   CI/CD pipeline before promotion.
5. **Deploy** — IaC-driven blue/green or canary release into the tenant's stamp.
6. **Scale** — Observability + FinOps monitor the tenant in production; the Deployment
   Stamps pattern lets the platform add capacity or spin up a new silo without redesign.

## 5. Environments and change management

| Environment | Subscription | Change path |
|---|---|---|
| Dev | Non-Prod | Direct pipeline deploy, no approval gate |
| Test / Staging | Non-Prod | Pipeline deploy with automated validation gate |
| Prod | Production | Pipeline deploy with mandatory approval + change record, IaC-only |

This closes the readiness pack's "no formal change process or approvals" and "no separate
dev/test/prod" gaps directly.
