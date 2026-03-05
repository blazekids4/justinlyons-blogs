# Master Risk List for LLM Applications

This is a comprehensive risk list for LLM applications that explicitly separates **data inferencing risks** (i.e., risks tied to model API access, data traveling to/from the model datacenter, and data while being processed in the datacenter) from all other risk categories (application, agent, RAG, lifecycle, governance, etc.).

The "other risks" section is primarily grounded in the OWASP Top 10 for LLM Applications (2025).

---

## A) Data Inferencing Risks (Model API + Data In-Transit + Data In-Use in Datacenter)

**Scope:** Assumes model API access exists, and focuses on risks to the data path (client ↔ model endpoint) and data during inference processing inside the hosting environment.

### A1) API Access & Request/Response Exposure Risks

These are risks that stem from the fact that an application can call a model API and must send/receive payloads.

#### 1) Unauthorized API invocation / stolen API credentials

If an attacker obtains API keys/tokens, they can submit prompts containing sensitive data (or trigger the app to do so) and receive outputs, potentially at scale. This risk is often downstream-amplified by agent/tooling patterns (see "Excessive Agency" in OWASP).

#### 2) Over-privileged model access from application identities

Service principals / managed identities calling the model may be broader than needed, enabling inference calls from more places than intended (lateral movement → inference endpoint usage). This aligns with OWASP's emphasis on least-privilege and excessive permissions in agentic systems.

#### 3) Inference request/response logging leakage

Even if the model provider does not train on the data, your application stack (API gateways, reverse proxies, APM, WAFs, SDK debug logs) may capture prompt/response text, embeddings, tool outputs, or headers containing sensitive context. OWASP repeatedly highlights sensitive information disclosure as a core risk, including inadvertent exposure through application context and configuration.

#### 4) Multi-tenant / cross-session data exposure bugs

Incorrect session handling, caching, or correlation IDs can cause "wrong user gets wrong response" or prompt context mixing. This often shows up as a "sensitive information disclosure" failure mode in LLM apps.

#### 5) Replay and tampering of requests if integrity controls are weak

If traffic integrity protections are misconfigured (e.g., proxies terminating TLS incorrectly or weak client pinning assumptions), inference calls can be replayed or modified, producing unintended outputs or exposing confidential context.

### A2) Data "In Transit" Risks (Client ↔ Datacenter Hosting Model)

These are risks tied to network transport of inference inputs/outputs.

#### 1) Eavesdropping / man-in-the-middle (MITM)

If TLS is downgraded, terminated improperly, or intercepted at untrusted layers, prompts/responses can be observed. Inference payloads commonly contain PII, credentials, internal documents, system prompts, or tool results—making confidentiality especially high impact. OWASP's "Sensitive Information Disclosure" category underscores the breadth of sensitive content that can be exposed.

#### 2) Metadata leakage

Even when payload is encrypted, metadata (endpoint names, DNS, SNI, request sizes, timing, frequency) can reveal usage patterns, business activity, or "who is asking what, when."

#### 3) Cross-border / routing concerns

Network routing (peering, WAN, edge/CDN layers) can introduce compliance concerns if traffic traverses jurisdictions unexpectedly (even when encrypted).

#### 4) Downgrade / cipher suite weaknesses

Weak TLS versions/ciphers, misconfigured cert validation, or legacy proxies increase the probability of interception.

### A3) Data "In Use" Risks (While Being Processed in the Datacenter)

These are risks that exist while the model host is actively processing prompts/responses.

#### 1) Unauthorized access inside hosting environment

Operator access pathways, compromised control plane, or insider risk could expose in-flight prompts/responses. OWASP's "Supply Chain" and "Unbounded Consumption" sections include real-world threats against infrastructure and hosted environments (e.g., compromises that can expose sensitive data).

#### 2) Residual data exposure in memory / GPU memory

Inference processing can leave artifacts in system memory or GPU local memory. OWASP references "LeftOvers / GPU local memory leakage" style risks in supply chain/infrastructure context, which can translate into inference-time data exposure in some threat models.

#### 3) Side-channel leakage (timing / cache / resource contention)

Attackers may infer aspects of prompts, retrieved context, or model behavior through timing, token streaming patterns, or resource-use signals—especially in multi-tenant settings. OWASP notes side-channel attacks as a concern in the context of model/API exploitation and extraction.

#### 4) Inference-time data retention policies

If prompts/responses are retained for debugging, abuse monitoring, or quality evaluation, retention expands the exposure window. OWASP stresses transparency, retention clarity, and minimizing sensitive data entering systems that can later be disclosed.

#### 5) Data mixing with tool outputs / retrieved context at inference time

During orchestration, the model often receives not just user input but also: RAG snippets, tool results, system instructions, policies. Any of that can be exposed if inference-time isolation is weak. OWASP points out how sensitive information and system prompts can leak and be leveraged for further attacks.

### A4) What "Data Inferencing Risks" Are Not

To keep the boundary crisp: items like prompt injection, RAG poisoning, excessive agency, output handling, etc. can lead to data exposure, but they are not transport/processing risks; they are application/control risks. OWASP treats these as distinct vulnerability categories.

---

## B) All Other Risks (Non-Inferencing-Data Risks)

This section is a "master list" of everything else—primarily mapped to OWASP Top 10 for LLM Apps 2025, plus a few common enterprise control-plane additions that typically sit adjacent to OWASP.

### B1) Prompt & Instruction Risks

#### 1) Prompt Injection (direct + indirect)

User or external content manipulates the model into violating policy, exfiltrating data, or invoking tools in unintended ways.

#### 2) Jailbreaking / safety bypass

A form of prompt injection where the attacker drives the model to ignore safety constraints.

#### 3) Multimodal injection

Hidden instructions embedded in images or other modalities that alter behavior.

### B2) Sensitive Information Disclosure (Beyond "in-transit/in-use")

#### 1) Model/application discloses PII, secrets, confidential business info

Can happen via training data memorization, application context leaks, or retrieval leaks.

#### 2) Training data inference / membership inference / model inversion

Attackers attempt to reconstruct sensitive inputs or infer whether certain records were part of training.

#### 3) User-driven disclosure

Users paste secrets/PII into prompts; later exposure occurs in output or logs. OWASP explicitly calls out user education and sanitization.

### B3) Supply Chain & Dependency Risks

#### 1) Compromised third-party libraries / components / registries

Traditional dependency attacks, plus ML-specific package risks.

#### 2) Vulnerable or tampered pre-trained models / adapters

Backdoors, hidden behaviors, malicious model artifacts.

#### 3) Weak model provenance

Lack of strong authenticity guarantees for published models; impersonation and repo compromise risks.

#### 4) Licensing / Terms & privacy-policy risks

Legal and data-use policy changes can introduce compliance and downstream exposure.

### B4) Data & Model Poisoning (Integrity Risks in Training/Fine-tuning/Embeddings)

#### 1) Poisoned training or fine-tuning data

Introduces bias, backdoors, degraded performance, "sleeper agent" behaviors.

#### 2) Poisoned embedding / RAG corpora

Malicious documents inserted into knowledge stores influence downstream answers and actions.

#### 3) Malware in model artifacts (e.g., unsafe serialization)

Poisoning extends beyond data: model files themselves can be delivery vehicles.

### B5) Improper Output Handling (Downstream System Exploits)

#### 1) LLM output used unsafely in code paths

XSS, CSRF, SSRF, command injection, SQL injection, path traversal, RCE when outputs are passed to interpreters or privileged components without validation.

#### 2) Lack of context-aware encoding and validation

Rendering model output into HTML/JS/SQL contexts without proper encoding.

### B6) Excessive Agency (Agent/Tool Misuse)

#### 1) Too much functionality exposed to the agent

Tools exist that the agent doesn't need (delete, send, modify).

#### 2) Too much permission

Agent tools run with broad service identities instead of user-scoped least privilege.

#### 3) Too much autonomy

High-impact actions occur without human approval or deterministic policy enforcement.

### B7) System Prompt Leakage (Policy & Secret Exposure)

#### 1) Disclosure of system prompts / hidden instructions

Exposes internal rules, system design, filtering logic, sometimes secrets—then used to craft stronger attacks.

#### 2) Mistaken belief that system prompts are "security controls"

OWASP explicitly warns prompts should not be considered secret nor used as primary security enforcement.

### B8) Vector & Embedding Weaknesses (RAG-Specific Risks)

#### 1) Unauthorized access / leakage from embeddings

Embeddings can encode sensitive information; retrieval can disclose it.

#### 2) Cross-tenant context leakage

Shared vector stores can leak data between users/groups without partitioning and access controls.

#### 3) Embedding inversion attacks

Attackers recover original text from embeddings.

#### 4) RAG data poisoning

Malicious documents manipulate outputs.

### B9) Misinformation / Hallucinations / Overreliance

#### 1) Hallucinated facts and false confidence

Model outputs appear credible but are wrong.

#### 2) Overreliance by users and systems

Humans (or downstream automation) trust outputs without verification.

#### 3) Unsafe code generation / hallucinated packages

Can introduce vulnerabilities or malware via developers following suggestions.

### B10) Unbounded Consumption (Availability + Cost + Model Theft)

#### 1) Denial of service via request flooding / long inputs

Resource exhaustion and service degradation.

#### 2) Denial of wallet (cost harvesting)

Attackers drive up usage costs in pay-per-token environments.

#### 3) Model extraction / functional replication

Query-based theft or synthetic-data generation to clone behavior.

#### 4) Side-channel strategies aimed at model info

OWASP frames side-channels as part of the broader exploitation and extraction surface.

---

## C) Quick "Cheat Sheet" Mapping

### "Inference data risks" = Confidentiality of prompts/responses along the path

- **API access misuse** (who can call it, what identity, what gets logged)
- **In transit** (TLS, MITM, routing/metadata)
- **In use** (host isolation, memory/GPU residue, side-channels, retention)

### "Everything else" = Behavior, integrity, orchestration, and lifecycle risks

- **Prompt manipulation** (Prompt Injection)
- **Data exposure by behavior** (Sensitive Info Disclosure, System Prompt Leakage)
- **RAG security** (Vector/Embedding Weaknesses)
- **Agent action risks** (Excessive Agency, Improper Output Handling)
- **Lifecycle & dependencies** (Supply Chain, Data/Model Poisoning)
- **Reliability** (Misinformation/Overreliance)
- **Resource abuse** (Unbounded Consumption)

**Note:** All of these "other" categories are explicitly enumerated and described in the OWASP LLM Top 10 (2025) document.
