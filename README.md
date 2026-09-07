# Enterprise SaaS Architecture Repository

## Start here — what is this, in plain English?

**What this describes, in one sentence:** how to run this platform as a proper
enterprise-grade, multi-tenant product on Microsoft Azure — with a front desk that checks
IDs, security guards, separate locked rooms for each customer, fire exits, and a
back-office team that keeps the lights on — all built using Microsoft Azure's standard
blueprint for doing this properly.

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
  explanation: [Multi-tenancy strategy](docs/01-multitenancy-strategy.md).
- **WAF (Well-Architected Framework)** = Microsoft's quality checklist for cloud systems
  — covering security, reliability, cost, speed, and day-to-day operations. Think of it as
  a building inspector's checklist.
- **CAF (Cloud Adoption Framework)** = Microsoft's step-by-step playbook for how to
  *organise and roll out* a cloud project properly — who owns what subscription, what
  gets checked before something goes live, how permissions are handed out.
- Both rulebooks are used throughout so that if a security team, auditor, or procurement
  team asks "was this built to best practice?" — the answer is yes, and every answer below
  points to exactly where in this pack it's proven:
  [WAF & CAF mapping](docs/05-waf-caf-mapping.md).

### How to explain this to someone else in 60 seconds

> "This sets out how the product runs as a properly built cloud setup. One document and
> picture covers the app itself, one covers who manages it day-to-day, and one covers the
> security fence around it — all built to Microsoft's official best-practice rulebook, so
> it can stand up to a real security review. It also shows how multiple customers can
> safely share the same setup, and the step-by-step plan to get there."

### Where to go next

1. **Want the fuller (but still readable) version?** →
   [Executive summary](docs/00-executive-summary.md)
2. **Ready for the deep technical dive?** See the [reading order](#reading-order) below.

---

This describes the target-state architecture for an **agentic AI platform for the
insurance industry**, evolving into a **multi-tenant, enterprise-grade SaaS platform on
Microsoft Azure**, aligned to the **Azure Well-Architected Framework (WAF)** and the
**Microsoft Cloud Adoption Framework (CAF)**.

It sets out three architecture views — **Application**, **Platform**, and
**Networking + Security** — plus a combined view, each backed by a written design
document and an accompanying architecture diagram built with official Microsoft Azure
stencils.

## Repository structure

```
├── README.md                                    ← this file
└── docs/
    ├── 00-executive-summary.md                  Current state, target state, why these 3 views
    ├── 01-multitenancy-strategy.md              How the platform achieves multi-tenancy on Azure
    ├── 02-application-architecture.md           Application architecture (view 1)
    ├── 03-platform-architecture.md              Platform / landing zone architecture (view 2)
    ├── 04-networking-security-architecture.md   Networking + Security architecture (view 3)
    ├── 05-waf-caf-mapping.md                    WAF pillar + CAF methodology traceability
    └── 06-migration-roadmap.md                  Phased plan to the target state
```

Each document is accompanied by an editable architecture diagram covering the same view:
`01-application-architecture`, `02-platform-architecture`,
`03-networking-security-architecture`, and `04-combined-architecture` (a single-page view
combining all three).

## Reading order

1. [Executive summary](docs/00-executive-summary.md) — start here
2. [Multi-tenancy strategy](docs/01-multitenancy-strategy.md)
3. [Application architecture](docs/02-application-architecture.md) + diagram: *01-application-architecture*
4. [Platform architecture](docs/03-platform-architecture.md) + diagram: *02-platform-architecture*
5. [Networking & security architecture](docs/04-networking-security-architecture.md) + diagram: *03-networking-security-architecture*
6. Diagram: *04-combined-architecture* — everything on one page
7. [WAF & CAF mapping](docs/05-waf-caf-mapping.md) — traceability for audit/procurement conversations
8. [Migration roadmap](docs/06-migration-roadmap.md) — how to get there

