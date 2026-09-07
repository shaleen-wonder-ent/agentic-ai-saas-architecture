# 00 — Executive Summary

## Where the platform is today

This is a proven MVP: a 3-tier, agent-based insurance automation platform (React
frontend, FastAPI gateway, LangGraph agent runtime, MCP tool layer, governance plane,
async workers) running as **Docker Compose services co-located on a single Azure VM**.

| Dimension | Current state |
|---|---|
| Compute | 1 Azure VM, all services co-located, vertical scale only |
| Environments | 1 (no dev/test/prod separation) |
| Identity | No SSO; local application accounts; no Entra ID |
| Secrets | Environment variables on the VM |
| Networking | Public inbound endpoint, no WAF/gateway, no private networking |
| Data | PostgreSQL, Redis, MinIO, Qdrant co-located on the same host |
| Governance | No landing zone, no Azure Policy, no tagging standard, no formal change process |
| Resilience | No backup/restore test, no DR plan, no RTO/RPO, no load testing |
| Tenancy | Application code already partitions data by `tenant_id` across Postgres/MinIO/Qdrant — but the *infrastructure* has no tenant isolation, autoscaling, or noisy-neighbor protection |

Onboarding real customer data, selling into regulated insurance carriers, and standing up
to a security review, an audit, or a CISO conversation all require the enterprise-grade
foundations set out in this document — security and identity, governance, and
scalability/resilience.

## What this document sets out

This sets out the detailed target architecture, split into the three views engineering
and security stakeholders can sign off independently, plus one that shows how they fit
together:

1. **[Application architecture](02-application-architecture.md)** — how the LangGraph
   agent runtime, MCP tools, governance plane and data tier run as a multi-tenant SaaS
   workload on Azure Container Apps.
2. **[Platform architecture](03-platform-architecture.md)** — the Cloud Adoption
   Framework landing zone (management groups, subscriptions, policy) and the SaaS control
   plane services (tenant management, observability, FinOps) that operate the platform.
3. **[Networking & security architecture](04-networking-security-architecture.md)** — the
   hub-spoke network, zero-trust controls, and defense-in-depth design that satisfies the
   Security pillar requirements.
4. A **combined single-page diagram** (*04-combined-architecture*) overlaying all three
   for an executive/board-level walkthrough.

Every design decision is traceable to a specific **Azure Well-Architected Framework**
pillar and a specific **Cloud Adoption Framework** methodology stage — see
[05-waf-caf-mapping.md](05-waf-caf-mapping.md) for the traceability matrix.

## Target state at a glance

| Dimension | Current (MVP) | Target (enterprise SaaS) |
|---|---|---|
| Compute | 1 Azure VM | Azure Container Apps (zone-redundant, autoscaled, KEDA-scaled workers) |
| Environments | 1 | Dev / Test / Staging / Prod, each its own subscription-scoped landing zone |
| Identity | Local accounts | Microsoft Entra ID (External ID for tenants, Workload ID for services), Conditional Access |
| Secrets | Env vars | Azure Key Vault + Managed Identity, zero secrets in code |
| Networking | Public, no WAF | Azure Front Door Premium + WAF, hub-spoke VNets, private endpoints only |
| Tenancy model | Logical partitioning only | Hybrid pool/silo model with the Deployment Stamps pattern for enterprise tenants |
| Governance | None | CAF landing zone, Azure Policy, tagging, IaC-only change path |
| Resilience | None | Zone redundancy, tested backup/restore, documented RTO/RPO, load & chaos testing |
| Observability/FinOps | Prometheus/Grafana (dev-only) | Azure Monitor, Log Analytics, Microsoft Purview, Cost Management + chargeback |

