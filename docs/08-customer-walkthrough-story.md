# 08 — Customer Walkthrough (Storytelling Script)

A single reference you speak from, front to back, in a customer conversation. It follows a
deliberate arc: **where you are today → where this goes → the big idea (multi-tenancy) →
the three architecture views → how it all maps to Microsoft's frameworks → how Azure itself
recommends doing multi-tenancy.**

Each stage gives you: an **opener** (how to start the section out loud), the **points to
walk through**, which **diagram to show**, and a **bridge** into the next stage.

## The arc at a glance

| # | Stage | Who leads | Diagram to show |
|---|---|---|---|
| 1 | Where you are today, and where this is heading | You | *Current vs. target (this doc)* |
| 2 | Multi-tenancy — the big idea | You | *07-multitenancy-explained* §1–2 |
| 3 | Application architecture | **Your colleague** | *01-application-architecture* |
| 4 | Platform architecture | You | *02-platform-architecture* |
| 5 | Networking & security | You | *03-networking-security-architecture* |
| 6 | How it maps to WAF & CAF | You | *05-waf-caf-mapping* |
| 7 | How Azure recommends multi-tenancy | You | *07-multitenancy-explained* + *04-combined* |

> Tip: keep the **combined diagram (*04-combined-architecture*)** on screen as the "home"
> picture. Zoom into the relevant band as you move through stages 3–5, then zoom back out
> for stages 6–7.

---

## Stage 1 — "Here's what we understand you have today, and where you want to take it"

**Opener:** *"Before we show you anything, let us play back what we understand about where
you are today — so you can tell us if we've got it right."*

**What they have today (say it as strengths + honest gaps):**

- It's a **proven MVP** — a working, agent-based insurance automation platform: a React
  front end, a FastAPI gateway, the LangGraph agent runtime, the MCP tool layer, a
  governance plane, and async workers.
- Today it all runs as **Docker Compose services on a single Azure VM** — one box, one
  environment, scaled only by making the box bigger.
- Importantly, **the application already thinks in tenants** — every piece of data is
  already tagged and separated by `tenant_id` across Postgres, blob and vector storage.
- What's **missing is the enterprise foundation around it**: no SSO/Entra ID, secrets sit
  in environment variables, a public endpoint with no WAF, no dev/test/prod separation, no
  landing zone or policy, and no backup/DR/load-testing story.

**Where this is heading (the target, one line each):**

- Compute → **Azure Container Apps**, zone-redundant and autoscaled.
- Identity → **Microsoft Entra ID** with Conditional Access; secrets → **Key Vault** with
  Managed Identity, zero secrets in code.
- Network → **Front Door + WAF**, hub-spoke VNets, **private endpoints only**.
- Tenancy → a **hybrid pool/silo model** using the Deployment Stamps pattern.
- Governance → a **CAF landing zone**, Azure Policy, tagging, and IaC-only change.
- Resilience → zone redundancy, tested backup/restore, documented RTO/RPO.

**Bridge:** *"The single biggest idea that shapes everything after this is multi-tenancy —
so let's start there at a high level, and we'll come back to the 'how' in detail later."*

---

## Stage 2 — "First, the big idea: what multi-tenancy means here"

**Show:** *07-multitenancy-explained*, sections 1–2 (the wristband analogy and Pool vs. Silo).

**Opener:** *"Multi-tenancy just means many customers safely sharing one product. Here's the
mental model."*

**Points to walk through (keep it high-level — promise detail later):**

- **The wristband.** Every request carries a `tenant_id` — issued once at sign-in by Entra
  ID, carried on every hop, and checked again at the database itself. That's what keeps one
  customer's data away from another's.
- **Pool vs. Silo.** Most customers **share** the same running platform (Pool — cheapest,
  fastest to onboard). A few large or regulated customers get their **own dedicated copy**
  (Silo — full isolation, sold as a premium tier).
- **One blueprint.** Both are built from the **same reusable template** (a "Deployment
  Stamp"), so onboarding a new customer is a **pipeline run, not a redesign**.

**Bridge:** *"That's the idea. Exactly how we achieve it on Azure — the isolation at each
layer — we'll cover at the very end. For now, let's look at how the platform is actually
built, starting with the application itself."*

---

## Stage 3 — Application architecture (handed to your colleague)

**Show:** *01-application-architecture*.

**Opener (your hand-off line):** *"The application layer is where [colleague] lives and
breathes — I'll let them walk you through how the app actually works."*

**One-sentence framing before you hand over (so the room has context):**

- *"At a high level: the same code we run today gets re-hosted onto Azure Container Apps —
  the React front end, the FastAPI gateway, the LangGraph agent runtime, the MCP tools, the
  governance plane and the async workers — plus the AI and data layers behind them. [Colleague]
  will take it from here."*

> Then hand over. When they finish, take the room back with the bridge below.

**Bridge (when you resume):** *"So that's what the product does. Now — who runs the building
it lives in? That's the platform architecture."*

---

## Stage 4 — Platform architecture (you, point by point)

**Show:** *02-platform-architecture*.

**Opener:** *"This is the 'management office' of the whole estate — how it's organised, who's
allowed to change what, and the shared services that keep it running day to day. It follows
Microsoft's Cloud Adoption Framework."*

**Walk it top-to-bottom, point by point:**

1. **Management-group & subscription hierarchy (CAF landing zone).** At the top, a tenant
   root management group splits into **Platform**, **Landing Zones**, and **Sandbox**.
   Under Platform sit **Connectivity, Identity, and Management** subscriptions; under
   Landing Zones sit **Production** and **Non-Production**. *Why it matters:* platform-wide
   policy and networking are managed once, and a problem in the product's prod subscription
   can't reach identity or hub resources.

2. **The SaaS control plane** (modelled on your own internal "AI Asset Fabric"), five
   columns:
   - **Governance & Control** — control tower, guardrails, AI governance, immutable audit.
   - **Agent Lifecycle** — agent builder/playground, testing lab, use-case marketplace.
   - **Connectivity** — MCP gateway, tool connectors, API Management (the AI gateway), Event Grid.
   - **Administration** — tenant management/registry, admin config, per-tenant keys.
   - **Observability & FinOps** — Azure Monitor/Log Analytics, Purview, Cost Management.

3. **Foundational platform services (CAF).** The always-on baseline: **Azure Policy**
   (denies public IPs, enforces tags/SKUs), **Entra ID**, **Key Vault**, **Defender for
   Cloud**, **Container Registry**, and **IaC + CI/CD** as the *only* change path.

4. **Landing-zone workload services.** The concrete data, AI, integration and scale services
   the landing zone provisions into the spokes — **Azure SQL Database (Ledger), Redis, Blob;
   Azure OpenAI, Document Intelligence, Content Safety, AI Search;
   Service Bus, Event Hubs, Logic Apps, Web PubSub, Notification Hubs; Container Apps, AKS,
   Load Testing.** *Why it matters:* the platform owns their baseline (private endpoints,
   policy, tags), not individual teams.

5. **The platform lifecycle.** Every tenant/use-case follows the same repeatable path:
   **Select Use Case → Configure → Integrate → Validate → Deploy → Scale** — auditable, not
   a bespoke engagement per customer.

**Bridge:** *"That's how it's organised and operated. The obvious next question from any
security team is: what stops a stranger walking in? That's the networking and security view."*

---

## Stage 5 — Networking & security (you, point by point)

**Show:** *03-networking-security-architecture*.

**Opener:** *"This is the walls, doors, key-cards and cameras. It's a zero-trust, hub-spoke
design — nothing talks to anything over the public internet unless we've explicitly allowed it."*

**Walk it point by point:**

1. **Hub-spoke topology.** A central **Hub VNet** (Connectivity subscription) holds the
   shared security services; the workload runs in two spokes — an **App Spoke** and a **Data
   Spoke** — connected by private **VNet peering**. No public transit between tiers.

2. **The inbound (edge) path.** Every request goes **Front Door Premium + WAF → DDoS
   Protection → Private Link → API Management (internal mode) → Container Apps (internal
   ingress only)**. The public IP with no WAF is gone.

3. **The outbound (egress) path.** External model providers and Microsoft Graph are reached
   only through **Azure Firewall Premium with an FQDN allow-list** — every outbound call is
   an explicit, logged, reviewable rule, not an open path from code.

4. **Private endpoints — no public access to data.** Every PaaS data/AI service (Azure SQL
   Database, Redis, Blob, Key Vault, Service Bus, Event Hubs, ACR, Azure
   OpenAI, AI Search, Document Intelligence, Content Safety, Web PubSub) is reachable **only**
   via a private endpoint, enforced tenant-wide by **Azure Policy**.

5. **Identity & access.** **Entra ID Conditional Access** for every human sign-in;
   **Workload Identity Federation** for service-to-service (no stored secrets); **PIM** for
   just-in-time admin elevation; **Bastion** as the only interactive admin path;
   **NSGs + ASGs** on every subnet, deny-by-default.

6. **Detection, posture, response.** **Defender for Cloud** (posture + Defender for
   APIs/Containers/Key Vault/DNS/Storage), and **Microsoft Sentinel** (SIEM/SOAR) correlating
   Front Door/WAF, Firewall, Entra and APIM logs.

**Bridge:** *"None of these are ad-hoc choices — every one traces back to Microsoft's own
best-practice frameworks. Let me show you that traceability."*

---

## Stage 6 — "How all of this maps to WAF & CAF"

**Show:** *05-waf-caf-mapping*.

**Opener:** *"When your security, audit or procurement teams ask 'was this built to best
practice?', this is the answer — every decision is traced to a framework pillar and the exact
gap it closes."*

**The five Well-Architected Framework pillars (one line each):**

- **Security** — Entra ID + Key Vault, Front Door + WAF, private endpoints, Firewall egress,
  Defender, threat model.
- **Reliability** — zone-redundant Container Apps, tested backup/restore + RTO/RPO,
  Deployment-Stamp DR, load/chaos testing.
- **Cost Optimization** — elastic autoscaling (pay per consumption), per-tenant cost
  tagging + chargeback.
- **Operational Excellence** — IaC-only change, dev/test/staging/prod separation, mandatory
  approvals, Sentinel detection.
- **Performance Efficiency** — KEDA-scaled workers + Service Bus, APIM rate limiting,
  capacity (PTU) sizing before GA.

**The Cloud Adoption Framework stages (one line each):**

- **Strategy & Plan** — pool/silo as a commercial tier; a phased migration roadmap.
- **Ready** — the management-group/subscription hierarchy and hub-spoke network.
- **Migrate/Innovate** — re-platforming from VM/Docker Compose to Container Apps.
- **Govern** — Azure Policy, tagging, RBAC + PIM, Purview.
- **Manage** — Azure Monitor, Defender, Sentinel, FinOps.

**Bridge:** *"Which brings us back to where we started — multi-tenancy — but now the detailed
'how', the way Microsoft itself recommends achieving it."*

---

## Stage 7 — "Finally: how Azure recommends multi-tenancy is achieved"

**Show:** *07-multitenancy-explained* (deep sections) and the **combined diagram** for the
whole picture.

**Opener:** *"Earlier we gave you the mental model. Here's how we actually deliver it on
Azure — straight from Microsoft's multitenant SaaS guidance."*

**Walk the 'how', point by point:**

1. **The pool-to-silo spectrum.** Microsoft defines a spectrum from fully shared (**pool**)
   to fully isolated (**silo**), with hybrid in between. We recommend **hybrid**: Pool for
   standard customers, Silo for regulated/enterprise customers.

2. **The Deployment Stamps pattern.** One IaC-defined "stamp" (Container Apps environment +
   data services + Key Vault) is deployed **once for the shared pool, again per silo tenant,
   and again per region** — a single reusable module, not two codebases.

3. **Isolation at every layer** (this is the reassurance slide for their security team):
   - **Identity** — Entra ID External ID / tenant claim per customer.
   - **Gateway** — an APIM product + subscription key and rate-limit policy per tenant.
   - **Compute** — `tenant_id` enforced in every agent/tool call (pool); a dedicated
     environment (silo).
   - **Relational data** — schema-per-tenant **plus Azure SQL Row-Level Security** (pool);
     dedicated server (silo).
   - **Blob / vector** — container/collection per tenant, or dedicated account (silo).
   - **Secrets** — Key Vault per environment or per silo tenant, never shared.
   - **Messaging** — per-tenant Service Bus queues so no one starves another.
   - **Cost** — `tenant_id`/`tier` tags feeding Cost Management for chargeback.

4. **Tenant lifecycle as automation.** A **Tenant Management Service** drives
   **Provision → Configure → Activate → Operate → Suspend/Offboard** through IaC pipelines —
   onboarding is a pipeline run, offboarding is auditable.

5. **Noisy-neighbour & blast-radius controls.** APIM per-tenant limits + per-tenant Service
   Bus queues + ACA autoscaling (pool); physical separation (silo); defense-in-depth data
   isolation so a bug in one layer is never a cross-tenant leak.

**What this unlocks (close on the business value):**

- One reference architecture serves **both** the shared pool and every enterprise silo — no
  architecture fork.
- Onboarding becomes a **pipeline run**, not a manual project.
- Isolation is sold as a **commercial tier** (Standard vs. Enterprise) instead of
  re-architecting per deal.

**Closing line:** *"So — same product you have today, re-platformed onto an enterprise-grade,
multi-tenant Azure foundation: secure, governed, resilient, and ready for a CISO or audit
conversation. Where would you like to go deeper?"*

---

## Related material to have open

- Current/target detail — [00-executive-summary.md](00-executive-summary.md)
- Multi-tenancy detail — [01-multitenancy-strategy.md](01-multitenancy-strategy.md) · [07-multitenancy-explained.md](07-multitenancy-explained.md)
- Application — [02-application-architecture.md](02-application-architecture.md) · diagram *01-application-architecture*
- Platform — [03-platform-architecture.md](03-platform-architecture.md) · diagram *02-platform-architecture*
- Networking & security — [04-networking-security-architecture.md](04-networking-security-architecture.md) · diagram *03-networking-security-architecture*
- Framework mapping — [05-waf-caf-mapping.md](05-waf-caf-mapping.md)
- Migration roadmap — [06-migration-roadmap.md](06-migration-roadmap.md)
