# 04 — Networking & Security Architecture

**Diagram:** *03-networking-security-architecture*

## Purpose

Defines the **hub-spoke network topology** and **zero-trust / defense-in-depth security
controls** that address the Security pillar directly: secrets management, SSO, inbound
protection with a WAF, private routing for model and data traffic, vulnerability/posture
scanning, and a documented threat model.

## 1. Topology — hub-spoke landing zone

Following the [CAF hub-spoke network topology](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/hub-spoke-network-topology):

| Network | Subscription | Purpose |
|---|---|---|
| **Hub VNet** (`10.0.0.0/22`) | Connectivity | Azure Firewall Premium, Azure Bastion, VPN/ExpressRoute Gateway, Private DNS Resolver + zones, shared Log Analytics/Sentinel workspace |
| **App Spoke VNet** (`10.0.4.0/22`) | Production | `aca-infra` subnet (delegated, VNet-integrated Azure Container Apps environment, internal ingress only), `apim-subnet` (APIM deployed in internal/VNet-injection mode), optional `aks-nodes` subnet (AKS for vector store / GPU workloads) and VNet-integrated Azure Functions (scheduled jobs) |
| **Data Spoke VNet** (`10.0.8.0/22`) | Production | `private-endpoints` subnet carrying private endpoints for every PaaS data service |

Hub↔spoke connectivity is via **VNet peering**; there is **no transit through the public
internet** between application, gateway, and data tiers.

## 2. Edge path

```
Internet / tenant users
   → Azure Front Door Premium + WAF policy   (TLS termination, managed + custom WAF rules, custom domains)
   → Azure DDoS Protection Standard           (volumetric attack mitigation, applied at the public IP)
   → Private Link                             (Front Door origin reaches APIM's private endpoint — no public IP on APIM)
   → API Management (internal mode, in the App Spoke)
   → Azure Container Apps (internal ingress only)
```

This replaces a public IP with no WAF or gateway.

## 3. Egress path (outbound to AI providers)

Because the platform must call external LLM providers (the internal LLM marketplace,
Anthropic, OpenAI fallback) and Microsoft Graph (email intake), egress is **not** a flat
allow-all:

- **Azure Firewall Premium** in the hub enforces an **FQDN allow-list** — only the specific
  provider endpoints and the Microsoft Graph API are reachable; everything else is denied
  by default.
- All egress is logged to the shared Log Analytics workspace for audit and anomaly
  detection.
- This makes every external call an explicit, reviewable firewall rule instead of an open
  outbound path from application code.

## 4. Private endpoints — no public network access to data

Every PaaS data/AI service used by the application is reachable **only** via a private
endpoint in the Data Spoke:

- Azure SQL Database (Ledger)
- Azure Cache for Redis
- Azure Blob Storage
- Azure Key Vault
- Azure Service Bus
- Azure Event Hubs
- Azure Container Registry
- Azure OpenAI Service
- Azure AI Search (vector store)
- Azure AI Document Intelligence
- Azure AI Content Safety
- Azure Web PubSub

Public network access is explicitly disabled on each resource and enforced tenant-wide via
an Azure Policy assignment (deny public network access on PaaS), not left to individual
resource configuration.

## 5. Identity & access controls

- **Microsoft Entra ID** — Conditional Access (device/location/risk-based policies) for
  every human sign-in; **Workload Identity Federation** for every service-to-service call,
  eliminating stored secrets/keys.
- **Privileged Identity Management (PIM)** — just-in-time elevation for any standing access
  the build/ops team needs, replacing broad standing access for the build team.
- **Azure Bastion** — the only path for interactive admin access to any VM-based
  component (e.g., build agents), no public RDP/SSH.
- **Network Security Groups + Application Security Groups** on every subnet — explicit
  allow rules per workload, deny by default.

## 6. Detection, posture, and response

- **Microsoft Defender for Cloud** — subscription-wide Cloud Security Posture Management
  (CSPM) plus workload-specific plans: **Defender for APIs** (protects the APIM surface),
  **Defender for Containers** (ACA/AKS), **Defender for Key Vault**, **Defender for DNS**,
  **Defender for Storage**.
- **Microsoft Sentinel** — SIEM/SOAR ingesting Defender alerts, Front Door/WAF logs,
  Firewall logs, Entra ID sign-in logs, and APIM logs for centralized detection and
  response.
- **Vulnerability scanning** — Defender for Containers scans ACR images pre-deployment;
  Defender for Cloud's agentless scanning covers the rest of the estate.

## 7. Threat model note (open design consideration)

A **threat model for a RAG workload** is still an open item. This architecture's
contribution to that threat model:
- **Prompt injection / data exfiltration via RAG** — mitigated by the guardrails layer
  (policy/PII/content-safety checks on every agent output) plus per-tenant vector store
  partitioning, so a compromised prompt cannot retrieve another tenant's embeddings.
- **Model/data exfiltration via egress** — mitigated by the Firewall FQDN allow-list; no
  arbitrary outbound destination is reachable from the agent runtime.
- **Credential theft** — mitigated by Workload Identity Federation (no long-lived API keys
  in code) for Azure-native calls; third-party LLM API keys remain in Key Vault with
  Managed Identity access only, rotated on a defined schedule.
- A **formal STRIDE-based threat model** for the RAG/agent pipeline specifically (beyond
  network-level controls) is recommended as an early workstream in the migration roadmap
  — see [06-migration-roadmap.md](06-migration-roadmap.md) — and should feed the
  **pen-test scope**, which remains an open item.

## 8. Private networking vs. developer velocity

This is an open design consideration. Recommended resolution: keep **prod and staging
fully private** (as described above); allow **dev** to use APIM in external/public mode
(still behind Entra ID auth and Front Door) so engineers iterate without a VPN/Bastion hop
for every change, with an Azure Policy exception scoped only to the Non-Production
subscription. This is a deliberate, documented exception — not an ungoverned gap.

---

**Next:** [WAF & CAF mapping](05-waf-caf-mapping.md)
