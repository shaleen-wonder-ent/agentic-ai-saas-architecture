# 02 — Application Architecture

**Diagram:** *01-application-architecture*

## Purpose

Defines how the platform's existing application components (React frontend, FastAPI gateway,
LangGraph agent runtime, MCP tool layer, governance plane, async workers) are re-hosted as
a **multi-tenant SaaS workload** on Azure PaaS, preserving the current codebase's
ports-and-adapters design (the same codebase runs on local Docker Compose or Azure
managed services).

## Layers (top to bottom in the diagram)

### 1. Tenant / client layer
Browser and API clients for pooled tenants (A, B, N) and one or more enterprise/silo
tenants, plus the platform admin persona used for control-plane operations.

### 2. Edge & identity (global)
- **Azure Front Door Premium + WAF** — single global entry point, TLS termination, custom
  domain per tenant if required, integrated DDoS Protection.
- **Azure API Management** acting as the **AI Gateway**: authenticates every request
  (Entra ID token validation), applies **per-tenant throttling/quota**, routes to the
  correct backend, and logs every request/response for audit.
- **Microsoft Entra ID** — External ID for tenant end-users, Workload Identity for
  service-to-service calls, Conditional Access policies.

This directly replaces the MVP's public inbound endpoint (no WAF or gateway) with a Front
Door + WAF, restricted-inbound design.

### 3. Application runtime — Azure Container Apps environment
One ACA environment per deployment stamp (shared for pool tenants, dedicated for silo
tenants), zone-redundant, with dev/test/staging/prod as separate environments/subscriptions
(see [03-platform-architecture.md](03-platform-architecture.md)):

| Component | Azure hosting | Notes |
|---|---|---|
| Frontend (React 19 SPA) | Azure Static Web Apps (or ACA if server-side rendering is needed) | Replaces the Vite dev server |
| Backend API Gateway | ACA, HTTP-scaled | FastAPI + Uvicorn; JWT/RBAC validated against Entra ID; per-tenant rate limiting middleware retained |
| LangGraph Agent Runtime | ACA, KEDA-scaled | Orchestrator + shared/use-case agents; unchanged business logic |
| MCP Tool Layer | ACA | claims / document / kb / fraud / compliance / notify tools; tool-scope enforcement startup invariant retained |
| Governance Plane | ACA | Hash-chained audit ledger, guardrails (policy/PII/content-safety checks), HITL queue |
| Async Workers | **ACA Jobs**, KEDA-scaled on Azure Service Bus queue depth | Replaces `arq` on Redis for the production target; email intake, RFI timers, webhooks, alerts |

**Tenant isolation:** `tenant_id` partitioning + Azure SQL row-level security for pooled
tenants; a dedicated ACA environment and database for enterprise/silo tenants, both
generated from the same Deployment Stamps IaC module (see
[01-multitenancy-strategy.md](01-multitenancy-strategy.md)).

### 4. Messaging, integration & eventing
**Azure Service Bus** replaces Redis-as-queue for production: queues/topics for
email-intake, agent-jobs, and notifications, and is the KEDA scale trigger for async
workers. Redis is retained only for cache/idempotency/rate-limit counters (see data layer).
Alongside Service Bus, the platform uses a broader integration/eventing set:

| Service | Role |
|---|---|
| **Azure Service Bus** | Queues/topics for email-intake, agent-jobs, notifications; KEDA scale trigger |
| **Azure Event Hubs** | High-volume streaming telemetry ingestion |
| **Azure Logic Apps** | Low-code integrations with partner/broker and line-of-business systems |
| **Azure Web PubSub** | Real-time push of pipeline updates to the console (complements SSE/WebSocket) |
| **Azure Notification Hubs** | Alert and email fan-out |
| **Azure Functions** | Scheduled jobs (RFI timers, polling, housekeeping) |

### 5. AI / model layer — Azure AI Foundry
- **Azure OpenAI Service**, accessed over a **private endpoint**, as the Azure-native LLM
  option.
- **Azure AI Document Intelligence** — OCR and structured field extraction for the
  Document Intelligence pipeline (ACORD forms, loss reports, scanned attachments).
- **Azure AI Content Safety** — content checks feeding the guardrails on every agent output.
- **Azure AI Search** — the vector index / knowledge store (see data layer).
- **The internal LLM marketplace** — remains the primary LLM, reached via APIM-managed
  egress rather than a raw outbound call.
- **Anthropic Claude** and **OpenAI (gpt-4o-mini) fallback** — external providers, reached
  through **controlled egress** (Azure Firewall FQDN allow-list — see
  [04-networking-security-architecture.md](04-networking-security-architecture.md)), not a
  direct internet path from application code.

### 6. Data layer — private endpoints only
| Current (MVP) | Target (Azure) |
|---|---|
| PostgreSQL 16 (Docker) | **Azure SQL Database (Ledger)** — tenant metadata, users, workflow state, audit (tamper-evident ledger tables), configurations; schema/RLS per tenant |
| Redis 7 (Docker) | **Azure Cache for Redis** — cache layer, job idempotency markers, rate-limit counters, token cache |
| MinIO | **Azure Blob Storage** — documents, attachments, processed files; container-per-tenant, immutability policy on audit artefacts |
| Qdrant | **Azure AI Search** — vector index, semantic search, embeddings; per-tenant collection |

All of these are reachable **only via private endpoints** inside the data spoke VNet — no
public network access for model or data traffic.

### 7. Cross-cutting platform services
Azure Key Vault (secrets/keys/certs, referenced via Managed Identity — no credentials in
code), Managed Identity, Azure Monitor/Application Insights/Log Analytics (replacing
Jaeger/Prometheus/Grafana dev tooling), Microsoft Purview (data classification/lineage/DLP
over the insurance PII the platform ingests), and Microsoft Defender for Cloud (workload
posture + Defender for APIs/Containers).

## Key architecture patterns carried forward unchanged

These patterns already exist in the codebase and are **preserved, not replaced**, by this
architecture:
- Agent-to-Agent (A2A) handoffs with typed contracts
- Model Context Protocol (MCP) typed tool layer
- SSE + WebSocket streaming to the console
- Guardrails on every agent output
- Immutable, hash-chained audit ledger (persisted in Azure SQL Database ledger tables — tamper-evident)
- Use-case marketplace (new workflows deployable without platform redeploy)
- Ports/adapters pattern (same code, different infrastructure bindings)

## Why Azure Container Apps rather than AKS

ACA + KEDA is the recommended production target. This is kept because:
- The workload is a set of stateless HTTP services and event-driven jobs — ACA's serverless
  Kubernetes-based model (KEDA, Dapr-ready, per-app scaling) fits without the operational
  overhead of running/patching an AKS control plane.
- ACA supports VNet integration, internal-only ingress, and per-environment isolation,
  satisfying the networking requirements in this pack.
- If a future requirement needs custom node pools, GPU nodes, or finer scheduling control,
  the same container images and Bicep/Terraform modules migrate to AKS with minimal
  rework — this is captured as an open design consideration in
  [06-migration-roadmap.md](06-migration-roadmap.md).

---

**Next:** [Platform architecture](03-platform-architecture.md)
