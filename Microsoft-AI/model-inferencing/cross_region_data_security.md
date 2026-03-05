# Data Privacy & Security: When Using Azure Direct Models Across Regions

**Microsoft Foundry (Azure AI Foundry)** | Customer Briefing | March 2026

---

## Scenario

An organization has all Microsoft Foundry resources — apps, data stores, agents, orchestration — deployed in **Region A**, but must use an Azure Direct Model (e.g., Azure OpenAI GPT-4o) deployed in **Region B**, located in a different country, due to model availability constraints.

**Primary concern:** Is sensitive data exposed, stored, or subject to foreign jurisdiction when inference runs outside the organization's home region?

---

## Executive Summary

Even when an Azure Direct Model runs in another region or country:

- ✅ Customer prompts and responses are **NOT** used to train models
- ✅ Data is encrypted in transit and at rest (Azure platform standard encryption applies)
- ✅ Data is not retained by OpenAI or shared across tenants
- ✅ Microsoft acts as the data processor under Azure's contractual framework (DPA)
- ✅ Optional controls exist to further restrict storage, logging, and review
- ✅ No persistent customer data is stored in the model itself
- ✅ Data stored at rest always remains in the customer's designated Azure geography

> **Key Principle:**  
> Cross-region inference does not mean cross-region data ownership or loss of control. The location of computation is distinct from the location of data storage and the governing legal framework.

---

## Deployment Type Determines Where Processing Occurs

This is the most important clarification for cross-region architectures. Microsoft offers three deployment types with different geographic processing guarantees:

| Deployment Type | Where Inference Processing Occurs |
|----------------|----------------------------------|
| **Standard / Regional** | Single region only — processed in the deployment region. No cross-region routing. |
| **DataZone** | Anywhere within a defined zone (e.g., all EU member nations, or US only) |
| **Global** | Any geography worldwide where the relevant model is deployed |

**Note:** Not all models support all deployment types. Options span Standard, Provisioned, and Batch variants within each processing category. Check Microsoft Foundry model availability documentation for current support by deployment type and region.

**Critical distinction:** For both Global and DataZone deployment types, any data stored at rest — including uploaded data and the abuse monitoring data store — remains in the customer-designated geography. Only the location of computation is affected; Azure data processing and compliance commitments remain fully applicable.

---

## How Data Flows in This Architecture

### What Happens During Inference

- The application in Region A sends a prompt to Azure Direct Models (Foundry)
- The request is TLS-encrypted. If Private Endpoints are configured, traffic traverses the Microsoft Azure backbone and avoids the public internet. Public endpoints are also TLS-encrypted but use the public-facing Azure endpoint.
- The model in Region B processes the request in memory (stateless — no data written to disk)
- The response is returned immediately to Region A

### What Does NOT Happen

- Data is **not** stored inside the model — models are stateless; prompts and completions are not used to train, retrain, or improve base models
- Data is **not** made available to OpenAI or other Azure Direct Model providers, and is not used to improve their models or services
- Data is **not** persisted outside the customer's Azure tenant unless stateful features are explicitly enabled
- Data is **not** shared with other tenants — strict tenant isolation is enforced at the compute layer

---

## Data Residency vs. Inference Location

These are distinct concepts that must be clearly separated for regulatory and legal analysis:

| Concept | Definition |
|---------|------------|
| **Data Residency** | Where data is stored at rest |
| **Inference Location** | Where GPU computation occurs |
| **Data Sovereignty** | Which legal framework governs the data |

Data stored at rest — including uploaded data — is always stored within the same geography as the customer's Foundry resource, regardless of deployment type. The inference location does not determine where data lives or which legal framework governs it.

---

## Data Retention & Storage

By default, prompts and responses are processed transiently. Data is stored only when optional features are explicitly enabled, such as the Responses API, Threads (Assistants API), Stored Completions, fine-tuning, or batch processing.

**When data is stored:**

- Stored at rest in the Foundry resource within the customer's Azure tenant, within the same geography as the resource
- Encrypted at rest by default using Microsoft-managed keys, with the option of customer-managed keys (CMK). Microsoft documents AES-256 as the standard for Azure data at rest.
- Can be deleted by the customer at any time
- Not available to OpenAI or used to train any generative AI foundation models

---

## Abuse Monitoring & Logging

The abuse monitoring system employs algorithms and heuristics to detect indicators of potential abuse. Automated review does not store prompts or completions and does not use them to train AI models or other systems.

**When human review is triggered:**

- The abuse monitoring data store is logically separated by customer resource
- A separate data store exists in each geography where the Azure Direct Model is available
- Customer prompts and content are stored in the Azure geography where the customer's Foundry resource is deployed — within the Azure Direct Models service boundary
- For Azure Direct Models deployed in the European Economic Area, the authorized Microsoft employees conducting human review are located in the EEA

Customers can apply to disable content logging entirely (modified abuse monitoring). When approved, the data storage and human review process is not performed — automated review may still run in-memory only. This status is verifiable via Azure resource properties (ContentLogging attribute).

---

## Threat Model & Technical Mitigations

The following table maps each relevant threat scenario to the specific technical controls Microsoft has implemented. This is intended for security teams performing risk assessment or vendor due diligence.

| Threat | Description | Microsoft Technical Mitigation | Concern Level |
|--------|-------------|-------------------------------|---------------|
| **Data Interception In Transit** | Prompt/response intercepted as it crosses regional or national boundaries over public infrastructure. | TLS 1.2+ enforced end-to-end across all endpoints. With Private Endpoints (Azure Private Link), traffic traverses the Microsoft backbone and eliminates public internet exposure. Without Private Link, TLS encryption protects confidentiality in transit over the public endpoint. | High |
| **Man-in-the-Middle (MITM) Attack** | Attacker intercepts and reads or modifies traffic between Region A and the model in Region B. | Certificate-based authentication prevents spoofing. Azure enforces strong cipher suites for TLS. Private Link removes the public attack surface entirely. (FIPS 140-2 compliant modules apply where Azure platform requirements mandate.) | High |
| **Nation-State Routing Attack / ISP Inspection** | Traffic routed through foreign infrastructure subject to government-compelled interception. | With Private Endpoints, traffic traverses the Microsoft backbone — bypassing public ISP routing entirely. Without Private Link, TLS encryption is enforced but traffic uses the public endpoint. Private Endpoints eliminate this risk vector. | High |
| **Data Persistence in Model (Cross-Region Leak)** | Prompts from one customer retained in model state and exposed to another region or tenant. | Models are stateless — prompts and completions are processed in memory and immediately discarded. No prompt caching, no replay, no reuse unless customer explicitly enables stateful features. | High |
| **Cross-Tenant Memory Access** | GPU memory from one tenant's inference leaking to another tenant's execution context. | Strict logical tenant isolation enforced at compute layer. Dedicated execution contexts. No shared prompt buffers across tenants. | High |
| **Microsoft Operator / Insider Access** | Microsoft engineer or cloud admin reads customer prompts during normal operations. | Human access is not part of normal operations. Requires JIT approval from team managers, Secure Access Workstations (SAWs), and point-wise queries by request ID only. Applies only to flagged content. | High |
| **Abuse Monitoring Data Storage** | Content flagged by abuse monitoring system stored in foreign geography, subject to local law. | Abuse monitoring data store is geographically bounded to the customer's Foundry resource geography. For EEA customers, human reviewers are located in the EEA. Customers can apply for modified monitoring to eliminate storage entirely. | Medium |
| **Unauthorized Training on Customer Data** | Customer prompts used to train or fine-tune models without consent. | Contractually prohibited and technically enforced. Azure DPA governs. No prompts or completions are used to train base models. Confirmed by Microsoft's data privacy documentation. | High |
| **Data at Rest Exposure (Stored Features)** | Data persisted via optional features (Threads, Responses API, etc.) stored unencrypted or in wrong region. | AES-256 encryption at rest by default. Optional customer-managed keys (CMK). All stored data remains in the customer's designated Azure geography regardless of inference location. | Medium |
| **Inference in Unacceptable Jurisdiction (Regulatory)** | Global deployment routes inference through a country with incompatible data protection laws. | DataZone deployment type bounds processing to a defined geographic zone (EU or US). Standard/Regional deployments stay within the single deployment region. Deployment type is customer-controlled. | Medium |

---

## Technical Deep Dive: Data Protection In Transit

### 1. Encryption In Transit

All communication between the customer application, Azure AI Foundry, and Azure OpenAI model endpoints is encrypted in transit. Azure enforces strong cipher suites. (For specific TLS version and cipher requirements, see the Azure encryption in transit documentation.)

- Strong cipher suites enforced by Azure platform policy
- Certificate-based authentication
- Applies regardless of region or country

### 2. Microsoft Private Global Backbone

When Private Endpoints (Azure Private Link) are configured, network traffic between the client and the service traverses the virtual network and Microsoft's Azure backbone — eliminating exposure to the public internet. Without Private Link, TLS encryption protects data in transit over the public endpoint, but the endpoint remains publicly addressable. For regulated environments requiring provable network isolation, Private Endpoints are the correct control.

### 3. Private Connectivity Options (Optional)

For higher-assurance environments, customers can use Private Endpoints (Azure Private Link), VNet-integrated Foundry resources, outbound traffic restrictions, and Network Security Perimeters. This ensures no public IP exposure, no inbound internet access, and explicit east-west traffic control.

---

## Technical Deep Dive: Data Protection While Processing

### 1. Stateless Model Execution

Azure Direct Models are stateless. Per Microsoft's Azure Direct Models data privacy documentation: the models do not store prompts or completions, and prompts and completions are not used to train, retrain, or improve the base models. Stored state occurs only when stateful features (Responses API, Threads, Stored Completions) are explicitly enabled.

### 2. Tenant Isolation at the Compute Layer

Azure Direct Models run in logically isolated environments with dedicated execution contexts, strict tenant boundary enforcement, and no cross-tenant memory access or shared prompt buffers. Even where GPUs are shared infrastructure, memory isolation is enforced by the platform.

### 3. Encryption at Rest

Any data that is stored is encrypted at rest using Microsoft-managed keys by default (customer-managed keys are optionally available), and is stored inside the customer's Azure geography. Customers can revoke keys and delete stored data at any time.

---

## Protection from Microsoft Operators & Insiders

Human access by Microsoft personnel is not part of normal operations. No engineer can browse customer data through operational dashboards. When triggered through abuse monitoring:

- Access requires Just-In-Time (JIT) approval from team managers (per Azure Direct Models privacy documentation)
- Authorized personnel use Secure Access Workstations (SAWs) (per Azure Direct Models privacy documentation)
- Access is point-wise by request ID — no bulk data access
- Applies only to content already flagged by the automated abuse monitoring system

Customers who require zero human access can apply for Modified Abuse Monitoring, which eliminates data storage and human review entirely.

---

## Highest Assurance: Confidential Inferencing

For customers with heightened regulatory or confidentiality requirements — or those who remain concerned about data exposure while in memory — Microsoft offers Confidential Inferencing (Preview).

**Confidential Inferencing ensures:**

- Data remains encrypted even while being processed
- Decryption occurs only inside hardware-based Trusted Execution Environments (TEEs)
- Neither Microsoft nor the hypervisor can access plaintext data

**Enforced using:**

- AMD SEV-SNP (CPU memory encryption)
- NVIDIA H100 Confidential GPU mode
- Hardware-backed remote attestation
- Key release is conditional on attestation — a common pattern in Azure confidential computing (e.g., Secure Key Release with Azure Key Vault), though specific implementation details of the Confidential Inferencing service should be verified against current Microsoft preview documentation

Before data is decrypted, the environment cryptographically proves that the correct firmware is running, the correct model container is loaded, and no unauthorized code is present — providing verifiable technical guarantees beyond contractual commitments. Note: Confidential Inferencing is a preview feature; specific implementation details including key management flow should be confirmed against current Microsoft preview documentation.

---

## Cross-Border Compliance & Legal Safeguards

Even when inference occurs in another country:

- Microsoft remains the data processor under the Azure Data Protection Addendum (DPA)
- Processing is covered by Standard Contractual Clauses (SCCs) where applicable
- Azure Direct Models operate under Azure's full compliance portfolio (ISO 27001, SOC 2, GDPR, etc.)
- Data remains subject to customer-controlled encryption and access policies
- Azure Direct Models do not interact with any services operated by OpenAI (e.g., ChatGPT or the OpenAI API)

---

## Private Endpoints for Cross-Region Inference

Both public and private endpoint architectures ultimately reach the same Azure Direct Model in Region B — and both benefit from TLS encryption, stateless model execution, and tenant isolation. The difference is not what the model does. The difference is who is allowed to reach it and how.

### Public vs. Private Endpoint: What Actually Changes

| Aspect | Public Endpoint | Cross-Region Private Endpoint |
|--------|----------------|------------------------------|
| **Network path** | Public Azure endpoint (publicly addressable IP) | Private IP inside customer VNet only |
| **Internet exposure** | Yes — endpoint is reachable from internet | No — eliminated entirely |
| **Public attack surface** | Exists: scanning, credential stuffing, API abuse | Eliminated: no reachable public ingress path |
| **Disable public access** | No | Yes — public endpoint can be fully disabled |
| **NSG / Firewall controls** | Limited — identity-based only | Full — apply NSGs, Azure Firewall, allow-lists |
| **Compliance posture** | Encrypted but publicly accessible | Private, isolated, not internet-accessible |
| **Model inference security** | Identical | Identical |
| **What changes** | Nothing inside the model | Who can reach the model and over what path |

> **Mental Model:**  
> **Public Endpoint:** We put a strong lock on a door that anyone can walk up to.  
> **Private Endpoint:** The building has no front door. You must already be inside our private network to reach it at all.

### Use Cases Where Private Endpoints Are ESSENTIAL

| Use Case / Industry | Why Private Endpoint Is Required | Governing Requirement |
|---------------------|----------------------------------|----------------------|
| **Financial Services (Banking, Capital Markets)** | Regulators mandate that AI systems processing customer financial data must not be internet-accessible. Public endpoint fails no-public-exposure audits even with encryption. | DORA (EU), FCA, OCC, MAS, APRA |
| **Healthcare & Clinical AI** | PHI processed through AI inference cannot traverse publicly addressable paths even if encrypted. HIPAA requires network controls beyond identity. | HIPAA Security Rule, HITRUST, NHS DSP Toolkit |
| **Government & Public Sector** | Government workloads require IL4/IL5 or equivalent network isolation. Public endpoints cannot satisfy no-internet-path requirements. | FedRAMP High, NIST SP 800-53 SC-7, IL4/IL5, UK OFFICIAL-SENSITIVE |
| **National Data Sovereignty (Cross-Border)** | Regulators increasingly require proof that cross-border AI traffic is not public. Private Endpoint provides the auditable network isolation required. | GDPR Article 32, Germany C5, Singapore PDPA advisory |
| **Zero Trust Architecture Mandates** | Zero Trust frameworks require microsegmentation and explicit allow-lists. Public endpoints cannot be microsegmented — private endpoints can. | NIST SP 800-207, CISA Zero Trust Maturity Model |
| **AI Handling Trade Secrets / IP** | Organizations processing proprietary R&D, legal strategy, or M&A data through AI require provable isolation — encrypted public is insufficient for legal and insurance purposes. | Cyber insurance requirements, attorney-client privilege, internal security policy |
| **Defense & Export-Controlled (ITAR/EAR)** | Export-controlled data processed through AI cannot use publicly addressable endpoints. This is a legal requirement, not just best practice. | ITAR, EAR, CMMC Level 2/3 |

### Use Cases Where Public Endpoints Are Acceptable

| Use Case / Profile | Why Public Endpoint Suffices | Typical Profile |
|--------------------|------------------------------|-----------------|
| **Consumer / SaaS Applications** | The app is already internet-facing. The AI endpoint shares the same exposure model as the rest of the application. No incremental risk from public endpoint. | SaaS companies, consumer apps, public-facing chatbots |
| **Internal Productivity Tools (Low Sensitivity)** | Prompts contain general business content, not PII, PHI, or trade secrets. Regulatory pressure is low. Entra ID authentication provides adequate control. | HR policy bots, IT help desk bots, non-sensitive knowledge search |
| **Developer / Prototype / Innovation Workloads** | Speed and simplicity matter more than compliance posture. No production data in scope. Security review not yet required. | Hackathons, innovation labs, proof-of-concept builds |
| **Low-Regulation Organizations with Strong Identity Controls** | Mature Entra ID policies, MFA, conditional access, and API key management provide adequate control. No sector-specific regulation mandates network isolation. | Lightly regulated tech companies with strong identity hygiene |
| **Latency-Sensitive Applications Accepting the Tradeoff** | Teams have assessed and accepted the risk tradeoff of public access in exchange for reduced routing latency and simpler architecture. | Real-time AI-assisted applications with low regulatory exposure |

### Network Controls Unlocked by Private Endpoints

Moving to a Private Endpoint architecture unlocks network-layer controls that are unavailable with public endpoints:

- **Network Security Groups (NSGs):** Restrict which specific subnets can initiate calls to the Azure Direct Model — all other subnets are blocked at the network layer, not just the identity layer
- **Azure Firewall integration:** All AI inference traffic can be routed through Azure Firewall for deep inspection, logging, and alerting — creating a full auditable trail of who called the model and when
- **Hub-and-spoke enforcement:** AI traffic is forced through a central security hub, enabling consistent policy enforcement across all workloads in the organization
- **Explicit network allow-lists:** Auditors and regulators can be shown a documented, enforceable list of approved network paths — not just identity controls
- **Disable public access entirely:** Once Private Endpoint is configured, the public endpoint can be fully disabled — eliminating the attack surface, not just hardening it

### The Assurance Escalation Ladder

For organizations deciding how much control is appropriate, the following levels represent progressive assurance from baseline to maximum:

| Level | Architecture | What It Protects | When to Use |
|-------|-------------|------------------|-------------|
| **1 — Baseline** | Public endpoint + TLS + Entra ID | Confidentiality in transit, identity-based access control | SaaS apps, prototypes, low-regulation workloads |
| **2 — Network Isolation** | Private Endpoint (cross-region) + disable public access | Eliminates public attack surface; enables network policy; satisfies no-internet-exposure mandates | Regulated industries, enterprise production, cross-border inference under regulatory scrutiny |
| **3 — Network + DataZone** | Private Endpoint + DataZone deployment type | Bounds inference to geographic zone (EU or US) AND removes public access | GDPR, national data sovereignty, financial regulatory compliance |
| **4 — Maximum Assurance** | Private Endpoint + DataZone + Confidential Inferencing + Modified Abuse Monitoring | Cryptographic proof that even Microsoft cannot read data in memory; no content logging; full network isolation | Sovereign AI, defense, highest-sensitivity regulated environments |

> **Key Principle for CISOs and Architects:**  
> Private Endpoints do not make the model safer. They make your access path provably private. That distinction matters far more to auditors, regulators, and architecture review boards — and it is what converts encrypted public access into network-isolated private access in a compliance framework.

---

## Bottom Line for Security & Legal Teams

Using an Azure Direct Model in another region:

- ✅ Does **NOT** expose data to OpenAI
- ✅ Does **NOT** allow Microsoft to reuse or train on customer data
- ✅ Does **NOT** persist prompts or responses by default
- ✅ Keeps stored data at rest in the customer's designated Azure geography, regardless of inference location
- ✅ Maintains full Azure-grade security, compliance, and contractual protections
- ✅ Can be further restricted with DataZone deployment types, Private Endpoints, modified abuse monitoring, or Confidential Inferencing

> **Executive Summary (One Sentence):**  
> Even when Azure Direct Model inference runs in another country, customer data is encrypted in transit, isolated per tenant, processed transiently in memory, and — when required — can be protected by hardware-based confidential computing that prevents even Microsoft from accessing it.

---

## Primary Microsoft References

- **Data, privacy, and security for Azure Direct Models in Microsoft Foundry** (Updated Feb 27, 2026)  
  [https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/openai/data-privacy](https://learn.microsoft.com/en-us/azure/foundry/responsible-ai/openai/data-privacy)

- **Azure AI Confidential Inferencing**  
  [https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/azure-ai-confidential-inferencing-preview/4248181](https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/azure-ai-confidential-inferencing-preview/4248181)

- **Azure Data Protection Addendum (DPA)**  
  [https://aka.ms/DPA](https://aka.ms/DPA)
