# Decentralized AI Agent Operating Model

## Case for Decentralized Model

Central teams struggle to scale agent development because domain context, data ownership, and integration complexity live in the BUs. Centralization increases cycle time and reduces correctness.

---

## FTE Breakdown for Decentralized AI Agent Operating Model (8 Business Units)

### Four Layers

#### 1. Central Platform Team — Govern

**Focus:** Governance, Security, Identity, Observability, Environments

- Owns non‑negotiable enterprise guardrails
- Enforces security, identity, observability, cost, and data boundaries
- Operates the AI control plane as a runtime system
- Does not build or own business agents

#### 2. AI Center of Excellence (CoE) — Enable

**Focus:** Standards, Guidance, Training, Reuse

- Owns standards, patterns, and reference architectures
- Scales expertise through training, reviews, and advisory
- Establishes evaluation and quality practices
- Enables teams without becoming a centralized delivery function

#### 3. BU AI Workload Teams — Execute

**Focus:** Domain Data, Agent Design, Integration, Lifecycle

- Own requirements, prioritization, and business outcomes
- Own domain data, correctness, and lifecycle operations
- Ensure solutions meet quality, security, and RAI expectations
- Act as accountable owners while following platform and CoE constraints

#### 4. Extended Developer Community — Accelerate

**Focus:** In‑Role Participation, No Additional Headcount

- Full‑time developers building agents within their existing roles
- Use standardized templates, patterns, and tooling
- Follow platform guardrails and CoE standards
- Scale delivery through peer‑to‑peer collaboration and reuse

---

## 1. Central Platform Team (Governance, Security, Identity, Observability)

**Recommended FTE: 14**

This team owns the enterprise AI control plane. Its scope is governance and enforcement — not application or agent delivery. The team remains relatively fixed in size as developer count grows because responsibilities do not scale linearly with usage.

### Roles

#### Platform Director (1)

- Owns the overall AI platform strategy and operating model
- Accountable for guardrails, enforcement posture, and platform maturity
- Sets priorities across security, identity, observability, and cost controls
- Serves as executive escalation point for platform‑level risk and decisions

#### Responsible AI (RAI) Lead (1)

- Owns enterprise Responsible AI strategy and policy implementation
- Defines RAI requirements, review thresholds, and escalation paths
- Coordinates with Legal, Risk, and Compliance stakeholders
- Ensures RAI controls are embedded into platform enforcement, not advisory

#### RAI / Compliance Specialist (1)

- Operationalizes Responsible AI and regulatory requirements
- Implements policy checks, documentation standards, and evidence collection
- Supports audits, reviews, and regulatory readiness
- Monitors adherence to enterprise RAI and compliance guardrails

#### Platform Engineers — Foundry / Copilot (3)

- Implement and maintain AI platform guardrails and integrations
- Enforce identity, environment, and tenant isolation patterns
- Operationalize policy enforcement through SDKs, APIs, and tooling
- Integrate evolving platform capabilities without owning business workloads

#### Platform Data / Knowledge Architect (1)

- Defines enterprise patterns for agent data access and grounding
- Owns data boundary enforcement, lineage expectations, and freshness models
- Establishes canonical approaches for RAG, retrieval, and knowledge access
- Partners with BU data stewards and CoE on data quality standards

#### AI Security Lead (1)

- Owns AI‑specific threat models and security control strategy
- Defines secure usage patterns for models, tools, and integrations
- Sets exception handling and escalation policies
- Partners with central security and platform engineering leadership

#### Security Engineers (2)

- Implement and operate AI security controls at runtime
- Monitor for misuse, abuse, and anomalous behavior
- Respond to incidents and tune security enforcement mechanisms
- Integrate new security capabilities as the threat landscape evolves

#### Observability Lead (1)

- Owns enterprise AI observability strategy and signal definition
- Defines required telemetry for agents, tools, models, and workflows
- Sets alerting standards for quality, safety, and reliability
- Ensures visibility into system behavior across all BUs

#### Observability / Monitoring Engineers (2)

- Build and maintain AI observability pipelines and dashboards
- Implement tracing, logging, and alerting for agent systems
- Detect drift, degradation, and emerging failure patterns
- Support incident response and continuous improvement loops

#### Cost / Quota Analyst (1)

- Owns AI usage visibility, cost attribution, and quota management
- Defines cost guardrails and budget thresholds across BUs
- Monitors consumption trends and flags anomalies
- Enables predictable scaling without constraining innovation

---

## 2. AI Center of Excellence (CoE) — Standards, Templates, Training, Advisory

**Recommended FTE: 12**

With 3,000 devs, the CoE must handle a much higher volume of training, advisory reviews, and enablement. CoE serves as the central expert group for training, patterns, standards, and reviews.

### Roles

#### CoE Director (1)

- Owns the enterprise AI enablement strategy and operating model
- Sets CoE priorities, success metrics, and engagement model with BUs
- Serves as executive escalation point for standards, quality, and risk tradeoffs

#### AI Strategy / Architecture Leads (2)

- Define enterprise‑wide AI and agent architecture patterns
- Translate business strategy into scalable technical approaches
- Review complex designs and guide BU teams on tradeoffs and decisions

#### Agent Orchestration / SDKs Experts (2)

- Define standard agent, tool, and orchestration patterns
- Produce reference implementations, templates, and SDK guidance
- Review BU agent designs to ensure conformance and best practices
- Do not build or own production workloads

#### Evaluation, Quality & Optimization Experts (2)

- Establish shared evaluation frameworks and quality standards
- Define metrics for accuracy, reliability, cost, and safety
- Review BU evaluation approaches and results
- Drive continuous improvement based on cross‑BU quality signals

#### Training & Adoption Leads (2)

- Design and run enterprise AI training and upskilling programs
- Maintain learning paths, documentation, and internal communities
- Support adoption through office hours, workshops, and playbooks
- Scale enablement across thousands of developers

#### CoE Advisors (3) — Dotted Line to BUs

- Embedded advisors supporting multiple BUs
- Provide hands‑on guidance during design and delivery phases

## 3. BU AI Workload Teams

**Recommended FTE: 5 per BU (8 BUs → 40 FTE total)**

BUs own:

- Requirements
- Domain data
- Flows & process integrations
- Lifecycle operations

Building can be done by distributed developers who follow the standards. This is the decentralized model: **Experts own correctness — devs do the bulk of the building.**

### Roles (per BU)

#### BU AI Lead / Product Owner (1)

- Owns BU AI vision, priorities, and roadmap
- Defines requirements and maps business processes to agents
- Accountable for end‑to‑end lifecycle (design → deploy → operate)
- Owns BU‑level observability, cost attribution, and budget management
- Primary interface between BU leadership, CoE, and platform teams

#### Domain Data Steward / Knowledge Engineering Lead (1)

- Owns BU domain data used for agent grounding and retrieval
- Curates, validates, and maintains domain knowledge sources
- Ensures alignment with platform data governance standards
- Responsible for retrieval quality, grounding accuracy, and data freshness
- Works closely with platform data roles on quality signals and guardrails

#### Agent Orchestration / SDKs Lead (1)

- Designs domain‑specific agent instructions and orchestration logic
- Owns prompt, tool, and workflow patterns specific to the BU
- Defines BU‑level evaluation scenarios tied to business outcomes
- Ensures agents follow enterprise patterns while reflecting domain nuance
- Acts as the technical authority for agent behavior correctness

#### Infra Deployment / Security Lead (1)

- Ensures compliant deployment of agents and integrations
- Validates tool usage, connectors, and external system access
- Enforces identity, security, and environment guardrails locally
- Coordinates with platform security and identity teams
- Owns BU‑specific deployment readiness and operational hygiene

#### QA / Model Evaluation / RAI Lead (1)

- Owns evaluation datasets, test coverage, and quality thresholds
- Oversees Responsible AI practices at the BU level
- Evaluates new models and model changes for BU suitability
- Monitors quality drift, safety issues, and regressions
- Acts as final quality gate before and after production changes

---s
Oversees Responsible AI practices at the BU level
Evaluates new models and model changes for BU suitability

## 4. Extended Developer Community - AI Champions Network

**No Additional Headcount**

This is a voluntary, opt‑in community of full‑time developers across the organization who lean in early on agentic AI and help scale adoption through peer influence. Participation occurs within existing roles and does not add new headcount or create a separate organization.

### Purpose

- Act as peer‑to‑peer multipliers for AI and agent best practices
- Accelerate adoption of approved patterns, templates, and tools
- Provide local mentorship and hands‑on support within teams
- Create fast feedback loops from real‑world usage back to the AI CoE

### Key Characteristics

#### Voluntary / Nominated

- Full‑time employees who opt in or are nominated based on interest and aptitude
- Participation reflects engagement, not a formal job change
- Membership may rotate over time as maturity grows

#### Peer‑to‑Peer Multipliers

- Share practical examples, lessons learned, and patterns with peers
- Support onboarding of additional teams into agent development
- Reinforce standards through collaboration, not enforcement

#### Not Accountable for Governance

- Do not define enterprise standards or guardrails
- Do not approve architectures or exceptions
- Operate within platform and CoE‑defined policies

#### Not Owners of Risk

- Do not own Responsible AI, security, or compliance decisions
- Do not act as quality or release gates
- Escalate concerns to BU champions, CoE, or platform teams as needed

---

## Consolidated FTE Model (8 Business Units)

### Centralized, Fixed‑Size Teams

| Team                              | FTE Count | Notes                                                  |
| --------------------------------- | --------- | ------------------------------------------------------ |
| **Central Platform Team**         | 14        | Scales by capability maturity, not developer count     |
| **AI Center of Excellence (CoE)** | 12        | Scales enablement and quality across ~3,000 developers |
| **Subtotal (Central)**            | **26**    |                                                        |

### Decentralized, Domain‑Owned Teams

| Team                     | FTE per BU | Number of BUs | Total FTE |
| ------------------------ | ---------- | ------------- | --------- |
| **BU AI Workload Teams** | 5          | 8             | 40        |

These roles are embedded in the business, accountable for outcomes, and aligned to domain priorities.

### Total FTE Summary

| Category                | FTE Count |
| ----------------------- | --------- |
| Central Platform Team   | 14        |
| AI Center of Excellence | 12        |
| BU AI Workload Teams    | 40        |
| **Total Formal FTE**    | **66**    |

### Extended Developer Community (No Additional Headcount)

**AI Champions Network (~60 developers)**

- Full‑time employees participating within existing roles
- Voluntary / nominated
- Not accountable for governance
- Not owners of risk
- Peer‑to‑peer multipliers for adoption and best practices

Not owners of risk
Peer‑to‑peer multipliers for adoption and best practices
This group scales delivery capacity without creating a shadow organization or increasing formal headcount.
