# 01 — Multi-Tenancy Strategy

This document answers the core question: **"How do we turn our product into a
multi-tenant SaaS product?"** It follows Microsoft's published guidance for
[Architecting multitenant solutions on Azure](https://learn.microsoft.com/azure/architecture/guide/multitenant/overview)
and the [SaaS design principles](https://learn.microsoft.com/azure/architecture/guide/saas/overview/introduction)
in the Azure Architecture Center, applied to the platform's existing codebase and data model.

## 1. What the platform already has (a head start)

The current codebase already implements several multi-tenancy prerequisites at the
**application layer**, per the platform architecture document:

- Per-tenant data partitioning enforced across PostgreSQL, MinIO (blob), and Qdrant
  (vector store) — every query is scoped by `tenant_id`.
- Per-tenant token-bucket rate limiting on run/playground endpoints.
- RBAC with fine-grained permission scopes enforced per API route and per MCP tool.
- An immutable, hash-chained audit ledger per tenant action.

What is missing is **infrastructure-level tenancy**: isolation, autoscaling, blast-radius
containment, and per-tenant lifecycle/billing — because everything currently runs on one
VM with no isolation boundary at all.

## 2. Tenancy model decision

Microsoft's guidance defines a spectrum from **fully multi-tenant (pool)** to **fully
isolated (silo)**, with **hybrid/mixed** models in between. We recommend a **hybrid model**
for the platform:

| Tier | Model | Rationale |
|---|---|---|
| Standard tenants (most customers) | **Pool** — shared Azure Container Apps environment, shared PostgreSQL Flexible Server with schema-per-tenant + row-level security (RLS), shared Redis, shared Blob Storage account with container-per-tenant | Lowest cost per tenant, fastest onboarding, matches the platform's existing `tenant_id` partitioning model |
| Regulated / enterprise tenants (large insurers, strict data-residency or compliance demands) | **Silo** — dedicated Azure Container Apps environment, dedicated database, dedicated Key Vault, optionally a dedicated region | Meets contractual isolation and residency commitments; contains blast radius; justified by deal size |
| Both tiers | **Deployment Stamps pattern** ([Microsoft guidance](https://learn.microsoft.com/azure/architecture/patterns/deployment-stamp)) — a "stamp" is a repeatable, IaC-defined unit (ACA environment + data services + Key Vault) that can be deployed per silo tenant or per region for scale-out | Gives a single reusable Bicep/Terraform module for both the shared pool and each dedicated silo, instead of two codebases |

This mirrors the platform's own "Multi-Tenant SaaS Platform" positioning as a stated
differentiator, and the stated production target (Container Apps + Service Bus + Entra
ID), while providing a sales-ready answer for enterprise prospects who require isolation.

## 3. Tenant isolation by layer

| Layer | Isolation mechanism |
|---|---|
| **Identity** | Microsoft Entra ID **External ID** tenant per customer organization (or app roles + tenant claim in a single Entra app, depending on final IdP decision), issuing a `tenant_id` claim consumed by APIM and the backend |
| **Edge / Gateway** | Azure API Management: one **APIM product + subscription key per tenant**, per-tenant rate limiting/quota policies, per-tenant request/response logging |
| **Compute** | Pool tier: single ACA environment, `tenant_id` propagated through every agent/tool call and enforced in MCP tool-scope checks. Silo tier: dedicated ACA environment (own IP range, own scaling, own outage domain) |
| **Data — relational** | Pool tier: PostgreSQL Flexible Server, **schema-per-tenant + Postgres Row-Level Security** as a defense-in-depth backstop to application-level scoping. Silo tier: dedicated Flexible Server instance |
| **Data — blob** | Container-per-tenant inside a shared Storage Account (pool) or dedicated Storage Account (silo); immutability policy on audit artefacts either way |
| **Data — vector** | Per-tenant collection/index in Azure AI Search or Qdrant, matching the existing Qdrant partitioning model |
| **Secrets** | One **Key Vault per environment** (pool) or **per silo tenant**, secrets referenced only via Managed Identity — never shared across tenant boundaries |
| **Messaging** | Azure Service Bus: per-tenant queues/topics or session-enabled queues keyed by `tenant_id`, so one tenant's backlog cannot starve another (noisy-neighbor control) |
| **Cost / billing** | Resource tagging (`tenant_id`, `tier`) feeding Azure Cost Management for **chargeback/showback**, consistent with the "FinOps" control-plane function in the platform architecture |

## 4. Tenant lifecycle (onboarding → offboarding)

A **Tenant Management Service** (new platform control-plane component — see
[03-platform-architecture.md](03-platform-architecture.md)) owns the tenant registry and
automates, via IaC pipelines:

1. **Provision** — create tenant record, Entra ID app role/tenant mapping, APIM
   product+subscription, Postgres schema (or dedicated stamp for silo), Blob container,
   Key Vault secret scope.
2. **Configure** — apply tenant-specific guardrail/policy configuration, use-case
   entitlements (which of the six use cases the tenant is licensed for), quota/throttling
   tier.
3. **Activate** — smoke test, enable in APIM, notify tenant admin.
4. **Operate** — per-tenant observability dashboards, per-tenant cost reporting, per-tenant
   SLA tracking.
5. **Suspend/Offboard** — revoke APIM subscription, disable Entra ID mapping, retain data
   per contractual retention period, then purge (schema drop / container purge) with an
   auditable record in the ledger.

## 5. Noisy-neighbor and blast-radius controls

- APIM per-tenant rate limiting + Service Bus per-tenant queues prevent one tenant's
  traffic spike from degrading others (pool tier).
- ACA autoscaling (HTTP + KEDA rules) absorbs legitimate bursts without manual
  intervention.
- Silo tier physically removes the shared-fate risk for tenants who require it.
- Defense-in-depth data isolation (RLS + application scoping + container-per-tenant) means
  a bug in one layer does not equal a cross-tenant data leak.

## 6. What this unlocks

- A single **reference architecture** (this repo's application/platform/networking
  diagrams) serves both the shared pool and, parameterized per stamp, every enterprise
  silo customer — no architecture fork.
- Tenant onboarding becomes a **pipeline run**, not a manual VM configuration exercise.
- Isolation can be quoted **as a commercial tier** (Standard/Pool vs Enterprise/Silo)
  instead of re-architecting per deal.
