# Enterprise SaaS Architecture Repository

## Start here — what is this, in plain English?

**The problem, in one sentence:** today the whole product runs out of a single computer
(one Azure VM) — like running an entire company out of one van. If that van breaks down,
gets a flat tyre, or someone breaks into it, the whole business stops. That's fine for a
first prototype, but no serious customer or auditor will trust real data to a "van."

**What we're proposing, in one sentence:** move the business out of the van and into a
proper building — with a front desk that checks IDs, security guards, separate locked
rooms for each customer, fire exits, and a back-office team that keeps the lights on —
built using Microsoft Azure's standard blueprint for doing this properly.

### The four pictures, explained like you're explaining it to a friend

| Picture | Plain-English question it answers | Building analogy |
|---|---|---|
| **Application architecture** | "What is the app actually made of, and how do its parts talk to each other?" | The floor plan of the shop itself — the till, the stockroom, the staff, how an order moves from front counter to warehouse |
| **Platform architecture** | "Who manages the building day-to-day — billing, monitoring, who's allowed to change what?" | The building's management office — reception, accounts department, maintenance team, the rulebook everyone follows |
| **Networking & security architecture** | "What stops a stranger walking in and stealing something?" | The walls, locked doors, key-cards, security cameras and guards around the building |
| **Combined architecture** | "Show me all of the above on one page." | A single site map showing the shop floor, the management office, and the security fence together |

### The other big ideas, in plain English

- **Multi-tenancy** = many customers share the same building safely. Most customers get
  their own locked *apartment* in a shared building (cheaper, faster to set up — this is
  called "pooled"). A few big customers who need extra guarantees can get their own
  *private house* instead (more expensive, fully separate — this is called "silo"). Full
  explanation: [docs/01-multitenancy-strategy.md](docs/01-multitenancy-strategy.md).
- **WAF (Well-Architected Framework)** = Microsoft's quality checklist for cloud systems
  — covering security, reliability, cost, speed, and day-to-day operations. Think of it as
  a building inspector's checklist.
- **CAF (Cloud Adoption Framework)** = Microsoft's step-by-step playbook for how to
  *organise and roll out* a cloud project properly — who owns what subscription, what
  gets checked before something goes live, how permissions are handed out.
- We used both rulebooks so that if a customer's security team, auditor, or procurement
  team asks "did you follow best practice?" — the answer is yes, and every answer below
  points to exactly where in this pack it's proven:
  [docs/05-waf-caf-mapping.md](docs/05-waf-caf-mapping.md).

### How to explain this to someone else in 60 seconds

> "We're moving the product off a single computer and into a properly built cloud setup.
> One document and picture covers the app itself, one covers who manages it day-to-day,
> and one covers the security fence around it — all built to Microsoft's official
> best-practice rulebook, so it can survive a real security review. We also show how
> multiple customers can safely share the same setup, and the step-by-step plan to get
> there from where we are today."

### Where to go next

1. **Still want the fuller (but still readable) version?** →
   [docs/00-executive-summary.md](docs/00-executive-summary.md)
2. **Ready for the deep technical dive?** See the [reading order](#reading-order) below.

---

This repository contains the target-state architecture for evolving an **agentic AI
platform for the insurance industry** from its current single-VM MVP into a
**multi-tenant, enterprise-grade SaaS platform on Microsoft Azure**, aligned to the
**Azure Well-Architected Framework (WAF)** and the **Microsoft Cloud Adoption Framework
(CAF)**.

It responds to the three architecture views the customer asked for — **Application**,
**Platform**, and **Networking + Security** — plus a combined view, each backed by a
written design document and an editable draw.io diagram built with official Microsoft
Azure stencils.

> **Note on this GitHub copy:** the `diagrams/` folder and the original source PDF/PPTX
> are intentionally excluded here (see `.gitignore`) because the diagrams and source
> documents still carry the customer's actual product/company names, and are shared
> directly by email instead. Everything under `docs/` is fully genericized and safe to
> host here.

## Source material reviewed

| Document | Content used |
|---|---|
| Platform technical architecture document (PDF) | Current MVP infrastructure/service inventory, tech stack, data flows, access-control design, stated production target |
| Enterprise-readiness gap-assessment deck (PPTX) | Gap assessment by pillar (Security, Governance, Scalability/Resilience), internal AI Asset Fabric reference architecture, customer rollout & deployment journey |

## Repository structure (this GitHub copy)

```
├── README.md                                    ← this file
├── .gitignore                                   excludes diagrams/ and the original PDF/PPTX
└── docs/
    ├── 00-executive-summary.md                  Current state, target state, why these 3 views
    ├── 01-multitenancy-strategy.md              How the platform achieves multi-tenancy on Azure
    ├── 02-application-architecture.md           Application architecture (view 1)
    ├── 03-platform-architecture.md              Platform / landing zone architecture (view 2)
    ├── 04-networking-security-architecture.md   Networking + Security architecture (view 3)
    ├── 05-waf-caf-mapping.md                    WAF pillar + CAF methodology traceability
    └── 06-migration-roadmap.md                  Phased plan from MVP to target state
```

Locally (not pushed here), the working folder also has a `diagrams/` directory with:
`generate_diagrams.py` (generator script, source of truth), and the four `.drawio` files —
`01-application-architecture.drawio`, `02-platform-architecture.drawio`,
`03-networking-security-architecture.drawio`, and `04-combined-architecture.drawio`
(a single-page view combining all three) — plus the original source PDF/PPTX.

## How to open the diagrams (local copy)

All four `.drawio` files use the standard **Azure shape library** (`mxgraph.azure.*`
stencils) that ships with [diagrams.net](https://app.diagrams.net) / draw.io desktop —
no extra shape libraries need to be installed. Open them directly in draw.io, VS Code's
Draw.io Integration extension, or app.diagrams.net.

If you change the architecture, edit `diagrams/generate_diagrams.py` and re-run
`python generate_diagrams.py` — this keeps all four diagrams (including the combined one)
consistent and versionable as code rather than hand-edited binary/XML blobs.

## Reading order

1. [docs/00-executive-summary.md](docs/00-executive-summary.md) — start here
2. [docs/01-multitenancy-strategy.md](docs/01-multitenancy-strategy.md)
3. [docs/02-application-architecture.md](docs/02-application-architecture.md) + [diagrams/01-application-architecture.drawio](diagrams/01-application-architecture.drawio)
4. [docs/03-platform-architecture.md](docs/03-platform-architecture.md) + [diagrams/02-platform-architecture.drawio](diagrams/02-platform-architecture.drawio)
5. [docs/04-networking-security-architecture.md](docs/04-networking-security-architecture.md) + [diagrams/03-networking-security-architecture.drawio](diagrams/03-networking-security-architecture.drawio)
6. [diagrams/04-combined-architecture.drawio](diagrams/04-combined-architecture.drawio) — everything on one page
7. [docs/05-waf-caf-mapping.md](docs/05-waf-caf-mapping.md) — proof of WAF/CAF alignment for the customer's auditors/CISO
8. [docs/06-migration-roadmap.md](docs/06-migration-roadmap.md) — how to get there
