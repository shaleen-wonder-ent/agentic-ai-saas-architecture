# Enterprise SaaS Architecture Repository

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
