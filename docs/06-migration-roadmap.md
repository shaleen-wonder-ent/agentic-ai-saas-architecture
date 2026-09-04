# 06 — Migration Roadmap: MVP → Enterprise SaaS

This phases the move from the single-VM MVP to the target architecture, sequenced against
the six-stage rollout journey already presented to customers
(Package → Deploy → Configure & Integrate → Validate → Go Live → Operate & Scale), and
explicitly surfaces the **open design considerations** the customer's own readiness pack
raised, so nothing is silently assumed.

## Phase 0 — Data pack (prerequisite, not yet complete per source pack)

Per the customer's own "still to be completed" list, these inputs are required before
sizing/capacity decisions in Phase 2 can be finalized:
- Current and projected users, requests, and peak load
- Model provider, endpoints, and current spend
- Data sources, classification, and PII exposure detail
- Target markets and residency constraints
- Compliance commitments already made to customers
- Team size, skills, delivery cadence
- Timeline and budget envelope

**Recommendation:** run this as a 1–2 week discovery workshop in parallel with Phase 1 —
it does not block landing zone setup, but it does block final capacity/SLA sign-off.

## Phase 1 — Foundation (landing zone + identity)

- Stand up the CAF management group hierarchy and the four subscriptions
  (Connectivity, Identity, Management, Prod, Non-Prod).
- Deploy the hub VNet (Firewall, Bastion, DNS) and baseline Azure Policy assignments.
- Stand up Microsoft Entra ID tenant configuration: app registrations, Conditional Access,
  PIM for the build team.
- Stand up Key Vault (per environment) and Azure Container Registry.
- Stand up the shared Log Analytics workspace + Microsoft Sentinel.

*Exit criteria:* an empty but policy-compliant, network-isolated landing zone that any
workload can be deployed into.

## Phase 2 — Application re-platforming

- Containerize the existing FastAPI/LangGraph/MCP/governance services (already Docker
  Compose-based, so this is a lift of existing Dockerfiles, not a rewrite).
- Deploy the Azure Container Apps environment (App Spoke), APIM (internal mode), Front
  Door + WAF.
- Migrate PostgreSQL → Azure Database for PostgreSQL Flexible Server (schema-per-tenant +
  RLS), Redis → Azure Cache for Redis, MinIO → Azure Blob Storage, Qdrant → Azure AI
  Search or managed Qdrant on AKS.
- Wire JWT/Entra ID production auth mode (already implemented in code per the PDF —
  "JWT (Production): RS256 JWT validated against a JWKS URL," just needs the Entra ID
  tenant wired and TLS/network isolation applied).
- Migrate `arq`/Redis async jobs to ACA Jobs + Service Bus + KEDA.

*Exit criteria:* the application runs end-to-end in the Non-Production subscription with
no public inbound endpoint and no secrets outside Key Vault.

## Phase 3 — Multi-tenancy, governance, and resilience

- Build the Tenant Management Service and onboard the first pooled tenants via pipeline
  (not manual configuration).
- Apply Microsoft Purview data classification/DLP to the ingested insurance PII (FNOL
  details, ACORD forms, claimant PII).
- Implement backup + tested restore for PostgreSQL/Blob; document RTO/RPO.
- Run the first load/soak test to establish real throughput limits (currently unknown per
  the readiness pack) and validate autoscaling rules.
- Run a formal STRIDE-based threat model workshop for the RAG/agent pipeline (flagged
  open in the readiness pack) and use its output to scope the pre-GA pen test (also
  flagged open).

*Exit criteria:* documented RTO/RPO, a load-tested capacity baseline, and a signed-off
threat model feeding the pen-test scope.

## Phase 4 — Enterprise hardening and scale

- Stand up Deployment Stamps automation so a silo tenant (dedicated ACA environment +
  database) can be provisioned by pipeline, not bespoke engineering.
- Evaluate a secondary region for DR based on the target markets/residency input from
  Phase 0; decide provisioned (PTU) vs. consumption pricing for Azure OpenAI based on the
  load-test output from Phase 3.
- Publish a customer-facing SLA once capacity, redundancy, and DR are validated.
- Complete the compliance position: which certifications are inherited from Azure vs. owned
  by the platform team, and the documented data-residency/retention statement.

*Exit criteria:* a publishable SLA, a documented compliance/residency position, and a
proven (pipeline-driven) path to onboard an enterprise/silo tenant.

## Sequencing rationale

Phases 1–2 remove the structural risks the readiness pack calls "consequences of running
everything in one VM" (single point of failure, blast radius, no environment separation,
change risk, evidence gap) as fast as possible. Phase 3 is what actually makes the platform
*multi-tenant* rather than just *re-hosted*. Phase 4 is what makes it *enterprise-sellable*
(SLA, DR, compliance story) rather than just technically sound.

## Open items carried forward (not resolved by architecture alone)

These require a business/commercial decision from the platform team, not just engineering:
- Sizing against projected peak volumes (needs Phase 0 data)
- Provisioned throughput vs. consumption pricing for LLM spend
- Quota and capacity confirmation in the target Azure regions
- The SLA the platform team is willing to publish to customers
- Landing zone pattern confirmation for a single-product estate (this pack proposes one —
  see [03-platform-architecture.md](03-platform-architecture.md) §1 — pending customer
  sign-off)
- Which compliance certifications are inherited from Azure vs. need to be independently
  obtained by the platform team
- Data residency options for each target market
- Final pen-test scope (depends on the Phase 3 threat model output)
