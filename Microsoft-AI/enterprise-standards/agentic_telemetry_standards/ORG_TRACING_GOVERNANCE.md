# Org-Wide Tracing Standard: Administrator Governance Guide

**Audience:** Platform Engineering · AI Observability Team · Architecture Review Board  
**Role:** Authors and custodians of the org-wide tracing standard  
**Purpose:** Define what is locked, what is org-governed, what developers may customize, and how to maintain the standard at scale across 3,000+ engineers.

---

## Table of Contents

1. [The Three Layers of Authority](#1-the-three-layers-of-authority)
2. [Layer 1 — What No One Can Change (OTel / GenAI Specification)](#2-layer-1--what-no-one-can-change)
3. [Layer 2 — What Administrators Own (The Org Contract)](#3-layer-2--what-administrators-own)
4. [Layer 3 — What Developers Control (Developer Namespace)](#4-layer-3--what-developers-control)
5. [Namespace Governance Model](#5-namespace-governance-model)
6. [When to Promote a Developer Attribute to the Org Contract](#6-when-to-promote-a-developer-attribute-to-the-org-contract)
7. [When NOT to Standardize an Attribute](#7-when-not-to-standardize-an-attribute)
8. [Cost Implications of Naming Decisions](#8-cost-implications-of-naming-decisions)
9. [Enforcement Mechanisms](#9-enforcement-mechanisms)
10. [Evolving the Standard: Change Management Process](#10-evolving-the-standard-change-management-process)
11. [SDK Upgrade & Spec Drift Policy](#11-sdk-upgrade--spec-drift-policy)
12. [Org Contract — Full Definition (Authoritative)](#12-org-contract--full-definition-authoritative)
13. [Administrator Checklist](#13-administrator-checklist)

---

## 1. The Three Layers of Authority

The tracing attribute space across this org is divided into three layers. Each layer has a distinct owner and a distinct set of rules for modification.

```
┌─────────────────────────────────────────────────────────────────────────┐
│            LAYER 1 — OPENTELEMETRY / GENAI SPECIFICATION                │
│  Owner: OpenTelemetry community & Azure SDK team                        │
│  Names: gen_ai.*, service.*, cloud.*, exception.*, telemetry.*          │
│  Admin authority: NONE — cannot be renamed or overridden                │
│  Change process: Track upstream SDK and spec releases                   │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
┌────────────────────────────────▼────────────────────────────────────────┐
│                  LAYER 2 — ORG CONTRACT                                 │
│  Owner: Platform Engineering / AI Observability Team (YOU)              │
│  Names: app.*, deployment.*, org.*, session.*, turn.*, run.*, user.*,   │
│         agent.*, thread.*, response.*                                   │
│  Admin authority: FULL — you define, version, and enforce these         │
│  Change process: RFC → Architecture Review → versioned release          │
└────────────────────────────────┬────────────────────────────────────────┘
                                 │
┌────────────────────────────────▼────────────────────────────────────────┐
│                  LAYER 3 — DEVELOPER NAMESPACE                          │
│  Owner: Individual teams / squads                                       │
│  Names: Any namespace not reserved by Layers 1 or 2                    │
│         (e.g., claim.*, policy.*, hr.*, inventory.*)                    │
│  Admin authority: GUIDELINES ONLY — you publish naming conventions,     │
│                   but teams own their domain attributes                 │
│  Change process: Team discretion; escalate to org contract if reused    │
└─────────────────────────────────────────────────────────────────────────┘
```

**Why this layered model matters:**

Unified dashboards and KQL queries depend on attribute names being consistent. A developer who renames `turn.latency_ms` to `latencyMs` or `duration_ms` silently breaks every latency chart across every team's workbook. The layered model makes the consequences of naming decisions explicit and traceable to an owner.

---

## 2. Layer 1 — What No One Can Change

### OpenTelemetry Semantic Conventions (spec-defined, SDK-emitted)

The following attribute names, span names, and metric names are defined by the [OpenTelemetry Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/) and the [OpenTelemetry GenAI Semantic Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/). They are emitted automatically by `AIAgentsInstrumentor` (`azure-ai-agents`) and by Microsoft Agent Framework.

**Administrator responsibility:** Do NOT define org-contract attributes that collide with these names. Monitor spec releases for new `gen_ai.*` attributes that may conflict with attributes you have already published in the org contract.

#### Fixed span names (SDK-emitted)

| SDK | Fixed span names |
|---|---|
| `azure-ai-agents` | `agents.create`, `agents.get`, `threads.create`, `messages.create`, `messages.list`, `runs.create`, `start_thread_run`, `get_thread_run`, `process_thread_run` |
| MAF | `invoke_agent {name}`, `chat {model}`, `execute_tool {fn}` |

#### Fixed attribute names (SDK-emitted, always present)

| Attribute | Defined by |
|---|---|
| `gen_ai.system` | OTel GenAI spec |
| `gen_ai.operation.name` | OTel GenAI spec |
| `gen_ai.request.model` | OTel GenAI spec |
| `gen_ai.response.model` | OTel GenAI spec |
| `gen_ai.usage.input_tokens` | OTel GenAI spec |
| `gen_ai.usage.output_tokens` | OTel GenAI spec |
| `gen_ai.agent.name` | OTel GenAI spec (MAF) |
| `gen_ai.agent.id` | OTel GenAI spec (MAF) |
| `gen_ai.token.type` | OTel GenAI spec (MAF metrics) |
| `service.name` | OTel resource spec |
| `service.version` | OTel resource spec |
| `service.instance.id` | OTel resource spec |
| `cloud.role` | Azure Monitor / OTel Azure resource spec |
| `cloud.roleInstance` | Azure Monitor / OTel Azure resource spec |
| `exception.type` | OTel general spec |
| `exception.message` | OTel general spec |
| `exception.stacktrace` | OTel general spec |
| `telemetry.sdk.name` | OTel core spec |
| `telemetry.sdk.version` | OTel core spec |
| `telemetry.sdk.language` | OTel core spec |

#### Fixed metric names (MAF, SDK-emitted)

| Metric | Defined by |
|---|---|
| `gen_ai.client.operation.duration` | OTel GenAI spec (MAF) |
| `gen_ai.client.token.usage` | OTel GenAI spec (MAF) |
| `agent_framework.function.invocation.duration` | MAF SDK |

### Reserved namespace prefixes (write-protect from org contract and developer use)

Administrators must ensure no org-contract or developer attribute uses these prefixes:

```
gen_ai.*       — OTel GenAI spec
service.*      — OTel resource spec
telemetry.*    — OTel core spec
cloud.*        — Azure Monitor / OTel cloud spec
exception.*    — OTel general spec
process.*      — OTel process spec
host.*         — OTel host spec
k8s.*          — OTel Kubernetes spec
http.*         — OTel HTTP spec
db.*           — OTel database spec
messaging.*    — OTel messaging spec
rpc.*          — OTel RPC spec
```

> **Practical rule:** If it starts with a known OTel prefix, it is spec-governed and off-limits for renaming or reuse. Check the [OTel Semantic Conventions registry](https://opentelemetry.io/docs/specs/semconv/) when in doubt.

---

## 3. Layer 2 — What Administrators Own

### The Org Contract: your authority and your responsibility

The org contract is the set of attribute names and span names that are **required across all teams** and that back the shared dashboards, KQL queries, Azure Workbooks, and alerting rules.

As an administrator, you:
- **Define** the attribute names (the contract below is authoritative)
- **Own the values** in `GlobalEnrichmentProcessor` (the shared `tracing_config.py` module)
- **Version** every change to the contract (see [Section 10](#10-evolving-the-standard-change-management-process))
- **Enforce** compliance via CI/CD lint gates (see [Section 9](#9-enforcement-mechanisms))
- **Document** every attribute's purpose, type, and value constraints in this file

### Reserved org namespaces

The following namespaces are **owned by the platform team**. No developer may create attributes in these namespaces without a contract RFC:

```
app.*          — service-level identity (app.service.name, app.version)
deployment.*   — environment identity (deployment.env)
org.*          — org-level dimensions (org.team, org.cost_center, org.product, org.squad)
session.*      — conversation session (session.id)
turn.*         — conversation turn (turn.number, turn.latency_ms)
run.*          — agent run (run.id, run.status)
agent.*        — agent identity (agent.id, agent.reused)
thread.*       — Foundry thread (thread.id)
user.*         — user identity (user.id, user.query — PII governed)
response.*     — assistant response (response.length)
experiment.*   — A/B testing (experiment.variant)
feature.*      — feature flags (feature.flag)
```

A developer who creates `org.my_custom_thing` in their code is violating the namespace boundary. Enforcement (see Section 9) should catch this at PR review time.

---

## 4. Layer 3 — What Developers Control

### The developer namespace: deliberately open

Developer teams are permitted — and encouraged — to add domain-specific attributes to their spans. This is where business context lives: claim IDs, policy numbers, intent classifications, routing decisions, confidence scores, etc.

**What administrators provide:**
- A list of reserved namespaces (Sections 2 and 3) that are off-limits
- Naming conventions developers must follow within their namespace
- A promotion path from developer attribute → org contract (Section 6)

**What administrators do NOT do:**
- Require approval for attributes in team-owned namespaces
- Pre-define every attribute a team may ever use
- Treat all developer attributes as candidates for standardization

### Developer namespace naming conventions to publish

Publish these as guidelines, not hard rules. They apply to any attribute not in a reserved namespace:

1. **Use dot-notation namespacing:** `claim.id`, `policy.number`, `hr.employee_id` — not `claimId`, `claim_id`, or `ClaimID`
2. **Use lowercase with underscores in the local part:** `claim.policy_number` — not `claim.policyNumber`
3. **Use noun phrases for identifiers, verb phrases for operations:** `claim.id` (identifier), `claim.validation_passed` (result of an operation)
4. **Keep cardinality in mind:** Attributes used as dashboard dimensions should have bounded cardinality (see [Section 8](#8-cost-implications-of-naming-decisions))
5. **Never put PII in attribute names** — only in values, and only where content recording policy permits
6. **Namespace by domain, not by team:** Use `claim.*` not `phoenix-team.*`. Teams change; domains persist.
7. **Boolean results:** Use the pattern `<domain>.<check_name>_passed` — e.g., `claim.fraud_check_passed: true`

### Explicitly permit these customizations

The following are always allowed without admin review:

| Customization | Example |
|---|---|
| Domain identifier attributes | `claim.id`, `policy.number`, `invoice.id` |
| Domain classification results | `claim.intent`, `claim.priority`, `policy.type` |
| Domain validation results | `claim.fraud_check_passed`, `policy.coverage_verified` |
| Domain size/count metrics | `claim.document_count`, `response.sentence_count` |
| Domain confidence scores | `routing.confidence`, `classification.score` |
| Workflow state | `claim.stage`, `approval.step` |

---

## 5. Namespace Governance Model

### Namespace ownership registry

Maintain a registry that maps each reserved namespace to its owner and the contract version it was introduced in. This is the source of truth for enforcement rules and CI/CD lint patterns.

| Namespace prefix | Owner | Contract version | Notes |
|---|---|---|---|
| `app.*` | Platform Engineering | v1.0 | Set via env vars in `GlobalEnrichmentProcessor` |
| `deployment.*` | Platform Engineering | v1.0 | Set via `ENVIRONMENT` env var |
| `org.*` | Platform Engineering | v1.0 | Set via env vars; `org.cost_center` mandatory for billing |
| `session.*` | Platform Engineering | v1.0 | `session.id` required on session root span |
| `turn.*` | Platform Engineering | v1.0 | All turn spans must carry `turn.number` and `turn.latency_ms` |
| `run.*` | Platform Engineering | v1.0 | Required on `agent_run` spans; `run.id` ties to Foundry Portal |
| `agent.*` | Platform Engineering | v1.0 | `agent.id` must match Foundry agent ID |
| `thread.*` | Platform Engineering | v1.0 | `thread.id` must match Foundry thread ID |
| `user.*` | Platform Engineering | v1.0 | `user.id` must be hashed in staging/prod (PII policy) |
| `response.*` | Platform Engineering | v1.0 | `response.length` is a proxy metric for output cost estimation |
| `experiment.*` | Platform Engineering | v1.1 | Added for A/B testing rollout |
| `feature.*` | Platform Engineering | v1.1 | Added for feature flag tracking |
| `gen_ai.*` | OTel GenAI spec | N/A — spec-governed | Reserved; never assign org attributes here |
| `service.*` | OTel resource spec | N/A — spec-governed | Reserved |
| `cloud.*` | Azure Monitor spec | N/A — spec-governed | Reserved |
| `exception.*` | OTel general spec | N/A — spec-governed | Reserved |
| `claim.*` | Claims AI squad | developer - not promoted | Domain namespace; team-owned |
| `policy.*` | Policy AI squad | developer - not promoted | Domain namespace; team-owned |
| *(all others)* | Originating team | developer | Available for team use |

### Namespace boundary rules for the lint gate

Express these as patterns in your CI/CD linter:

```
BLOCKED on any span:   gen_ai.*, service.*, cloud.*, exception.*, telemetry.*
BLOCKED outside org:   app.*, deployment.*, org.*, session.*, turn.*, run.*, agent.*, thread.*, user.*, response.*, experiment.*, feature.*
ALLOWED anywhere:      <team-domain>.*  (claim.*, policy.*, hr.*, inventory.*, ...)
```

---

## 6. When to Promote a Developer Attribute to the Org Contract

Promotion is the process of taking an attribute a developer team invented and elevating it to the org contract so all teams are required (or recommended) to set it.

### Promotion criteria — ALL must be met

An attribute should be promoted only when:

1. **Cross-team reuse.** At least two independent teams have independently created the same semantic attribute with the same (or near-identical) name and meaning.
2. **Dashboard value.** The attribute unlocks a dashboard tile or KQL query that is useful to managers or the platform team across services — not just within one team's context.
3. **Cardinality is bounded.** The attribute's value space is finite and manageable (e.g., an enum, a status string, an integer count). Unbounded strings (free-form text, user IDs without hashing) are not promotable to required fields.
4. **PII-free or policy-governed.** The attribute either contains no PII, or has a clear policy for PII handling that can be expressed in the contract (e.g., `user.id` must be hashed).
5. **Stable semantics.** The attribute's meaning is unlikely to change in the near future.

### Promotion criteria — any of these blocks promotion

- The attribute would duplicate a `gen_ai.*` spec attribute (e.g., promoting `my_org.input_tokens` when `gen_ai.usage.input_tokens` already exists)
- The attribute name conflicts with an existing or draft OTel semantic convention
- The attribute value is free-form text that could carry PII at runtime
- The attribute is specific to one team's internal workflow with no org-wide analogue

### Promotion process

```
1. PROPOSAL
   Team lead submits a promotion request to the platform team, including:
   - Proposed attribute name (must follow namespace rules)
   - Type, valid values or range, example value
   - Which teams currently use it and under what name
   - At least one KQL query or dashboard tile that depends on it
   - PII assessment

2. REVIEW (Architecture Review Board)
   - Does it conflict with Layer 1 (check OTel spec)?
   - Is the namespace appropriate for org ownership?
   - Cardinality assessment — will this inflate Application Insights costs?
   - Cross-team naming alignment — if two teams used different names,
     which becomes canonical? The non-canonical users must migrate.

3. DECISION
   - Approved → added to org contract, assigned to next contract version
   - Approved with rename → canonical name chosen, team must migrate
   - Rejected → attribute stays developer-defined; teams may continue using it
   - Deferred → flagged for re-evaluation when a second team adopts it

4. PUBLICATION
   - Attribute added to this document (Section 12) and ATTRIBUTE_REFERENCE.md
   - GlobalEnrichmentProcessor updated if it can be injected from env vars
   - New contract version tagged in the shared tracing package
   - Migration guide published for teams not yet using the attribute

5. ENFORCEMENT (next contract version release + 30 days)
   - CI/CD lint rule added to require the attribute on the target span type
   - Teams given 30-day migration window before lint rule becomes blocking
```

---

## 7. When NOT to Standardize an Attribute

Standardizing too aggressively creates three problems:
1. **Compliance burden** — teams spend time adding required attributes that add no value for their use case
2. **False precision** — attributes that mean different things in different domains appear identical in dashboards
3. **Cardinality cost** — unnecessary high-value required fields inflate Application Insights ingestion costs

### Attributes that should remain developer-defined

| Category | Why it should NOT be org-contracted | Example |
|---|---|---|
| Domain object identifiers | Specific to one domain; no cross-team join key | `claim.id`, `invoice.number` |
| Internal business state | Domain-specific state machines | `claim.stage`, `approval.workflow_step` |
| Model scores and probabilities | Highly domain-specific; different scales | `routing.confidence`, `fraud.score` |
| Intermediate computation results | No org-wide semantics | `embedding.dimension`, `chunk.index` |
| Debug-only context | Should be behind feature flags, not required | `prompt.template_version`, `retrieval.top_k` |
| Attributes unique to one team's architecture | Cannot be meaningfully generalized | `claims.legacy_system_code` |

**A useful test:** If a manager looking at a session-level dashboard for a *different* product would be confused by this attribute or would never filter by it, it should stay developer-defined.

---

## 8. Cost Implications of Naming Decisions

Application Insights bills on data ingestion volume. Attribute cardinality and span count both directly affect cost. As the administrator defining the org contract, you are the primary lever for controlling these costs across all products.

### High-cardinality attributes

High-cardinality attributes have a large number of unique values. In Application Insights, attributes ending up as `customDimensions` columns create per-row storage. Free-form text fields (prompt content, response text) are the biggest cost drivers.

| Attribute type | Cardinality | Cost impact | Admin guidance |
|---|---|---|---|
| Boolean (`true`/`false`) | 2 | Very low | Fine to require in org contract |
| Enum / status string | 3–20 values | Low | Fine to require |
| Integer count (tokens, turns) | Hundreds | Low–medium | Fine to require; prefer ranges in dashboards |
| Agent or thread ID (UUIDs) | Millions | Medium | Require in org contract — used for lookup, not aggregation |
| User ID (hashed) | Millions | Medium | Require in org contract — used for per-user analytics |
| User query text | Unbounded | **Very high** | **Development only.** Gate behind content recording env var. |
| Response text | Unbounded | **Very high** | **Development only.** Gate behind content recording env var. |

### Span count: polling suppression

The `get_thread_run` span from `azure-ai-agents` is emitted on every polling tick (~1/second). For a 10-second agent run, this produces 10 spans. At scale (1,000 sessions/day × 2 turns × 10 polls), that is **20,000 polling spans/day** that provide zero analytical value.

**Admin requirement:** The `PollingSuppressionProcessor` and `FilteringSpanExporter` in `tracing_config.py` must be included in every `azure-ai-agents` service. This is a cost control requirement, not optional.

**Estimated savings at scale:**

| Scale | Without suppression | With suppression | Monthly savings |
|---|---|---|---|
| 1,000 sessions/day | ~20,000 spans/day | ~2,000 spans/day | ~$40–80/month per service |
| 10,000 sessions/day | ~200,000 spans/day | ~20,000 spans/day | ~$400–800/month per service |
| 100,000 sessions/day | ~2M spans/day | ~200K spans/day | ~$4,000–8,000/month per service |

> MAF does not poll — this cost only applies to `azure-ai-agents`.

### Sampling rates by environment

The org contract specifies these sampling rates. Administrators should review and adjust annually based on actual ingestion costs.

| Environment | Sampling rate | Rationale |
|---|---|---|
| `dev` | 100% | All traces needed for debugging |
| `staging` | 100% | Full coverage for pre-release validation |
| `prod` (default) | 20% | Cost-controlled; error traces always kept via tail-based sampler |
| `prod` (regulated/compliance) | 100% | Some regulated industries require full audit trail |

> For large-scale production, strongly recommend implementing a **tail-based sampler** via an OpenTelemetry Collector. Tail-based sampling keeps 100% of error traces and sampled 20% of success traces — you never lose a failure even at low sampling rates.

---

## 9. Enforcement Mechanisms

### Automated enforcement (CI/CD lint gate)

Publish a linter check that runs on every PR against any AI agent service in the org. The linter should:

**Prohibit:**
- Any `set_attribute()` call where the key uses a Layer 1 reserved prefix (`gen_ai.*`, `service.*`, `cloud.*`, `exception.*`, `telemetry.*`)
- Any `set_attribute()` call where the key uses a Layer 2 org namespace but is not in the approved contract attribute list
- Any span name that deviates from the org-standard names (`session`, `turn-{N}`, `agent.{action}`) using a regex match
- Any span name containing a `/` character, a `.` in middle segments, or exceeding 60 characters

**Require:**
- That `setup_tracing()` (or `setup_tracing_maf()`) appears before any `AgentsClient` or `Agent(...)` instantiation in the entry point file
- That turn spans carry `turn.number` and `turn.latency_ms`
- That error handling follows the propagation pattern (both inner span and parent span marked `ERROR`)

**Warn (non-blocking):**
- `user.id` set to a value that appears to be an email address (regex: `.*@.*\..*`)
- `set_attribute("user.query", ...)` called in a file where `ENVIRONMENT` is not `dev`
- `SimpleSpanProcessor` used in production code (should be `BatchSpanProcessor`)

### Manual enforcement: PR review gate

In your PR review checklist, add:
- [ ] Does this service call `setup_tracing()` before any agent client instantiation?
- [ ] Are turn spans named `turn-{integer}` (not `Turn 1`, `turn_1`, `TURN-1`)?
- [ ] Are all org-contract attributes set on the correct span types?
- [ ] Is `user.id` hashed before being passed to `set_attribute()`?
- [ ] Does error handling propagate `StatusCode.ERROR` to parent spans?
- [ ] Is `BatchSpanProcessor` used instead of `SimpleSpanProcessor`?

### Dashboard-driven enforcement (reactive)

Set up Application Insights alerts that fire when expected org-contract attributes go missing:

```kusto
// Alert: sessions without org.team attribute (indicates tracing setup skipped)
dependencies
| where timestamp > ago(1h)
| where name == "session"
| where isempty(tostring(customDimensions["org.team"]))
| summarize missing_count = count() by bin(timestamp, 5m)
| where missing_count > 10
```

```kusto
// Alert: turn spans without turn.latency_ms (broken latency dashboard)
dependencies
| where timestamp > ago(1h)
| where name startswith "turn-"
| where isempty(tostring(customDimensions["turn.latency_ms"]))
| summarize missing_count = count() by tostring(customDimensions["app.service.name"])
| where missing_count > 5
```

These alerts notify the platform team, not the developer team — this is a systematic failure, not a code bug.

---

## 10. Evolving the Standard: Change Management Process

### Versioning the org contract

The org contract follows **semantic versioning** applied to the shared `tracing_config.py` package:

| Change type | Version bump | Examples |
|---|---|---|
| New required attribute | Minor (`1.0` → `1.1`) | Adding `experiment.variant` to the session contract |
| New optional attribute | Patch (`1.1.0` → `1.1.1`) | Adding `feature.flag` as a recommended field |
| Rename of existing attribute | Major (`1.x` → `2.0`) | Renaming `turn.latency_ms` → `turn.duration_ms` |
| Removal of attribute | Major (`1.x` → `2.0`) | Removing a deprecated attribute from the contract |
| Reserved namespace added | Minor | Reserving a new org prefix |

### Change categories and their processes

#### Non-breaking additions (minor version)

Adding a new attribute or reserving a new namespace. No existing code breaks.

```
1. RFC document (1 page): attribute name, type, purpose, which spans, who owns it
2. Architecture Review Board approval (async — 3 business days)
3. Update GlobalEnrichmentProcessor or add to per-span contract
4. Update this document and ATTRIBUTE_REFERENCE.md
5. Tag new package version
6. Notify all teams via #ai-platform Slack channel
7. 30-day window for teams to adopt (not blocking)
8. CI/CD lint rule added after adoption window
```

#### Breaking changes (major version)

Renames or removals. All teams must migrate. Rare — treat with extreme caution.

```
1. RFC document with full migration guide, including sed/grep search patterns
2. Architecture Review Board approval (synchronous meeting required)
3. Dual-emit period (6 weeks): GlobalEnrichmentProcessor emits BOTH old and new name
4. Dashboard updates: all KQL queries and workbooks updated to read new name
5. Alert updates: monitoring rules updated
6. Package tagged with new major version
7. Teams notified with 6-week migration deadline
8. After 6 weeks: old attribute name removed from GlobalEnrichmentProcessor
9. CI/CD lint rule blocks old attribute name
10. Post-migration report: confirm all services moved (query Application Insights)
```

> **Strongly discourage renames.** Every rename requires a dual-emit period, a dashboard migration, team migrations, and a lint rule change. Get the name right before promoting to the org contract.

### Backward compatibility commitment

Once an attribute is in the org contract, provide the following guarantees:
- Required attributes will remain supported for a **minimum of 12 months** after any deprecation announcement
- The attribute's type will not change without a major version bump
- The attribute's semantics (what it means) will not silently change without a changelog entry

---

## 11. SDK Upgrade & Spec Drift Policy

The OpenTelemetry GenAI semantic conventions evolve. New `gen_ai.*` attributes are added, and occasionally existing ones are renamed. The platform team must track these changes to:

1. **Prevent collisions** — if the spec adds `gen_ai.turn.number` and you already have an org contract attribute that means the same thing, you have a conflict.
2. **Take advantage of new conventions** — if the spec formalizes something you have in the developer namespace, promote your org attribute to the spec-defined name.

### Monitoring process

- Subscribe to the [OTel Semantic Conventions changelog](https://github.com/open-telemetry/semantic-conventions/releases) — review on every minor release
- Subscribe to `azure-ai-agents` and `agent-framework` release notes — SDK updates sometimes add new auto-instrumented attributes
- Quarterly: run a scan across Application Insights to identify attributes starting with `gen_ai.` that were not present 3 months ago (indicates a new SDK auto-attribute)

```kusto
// Detect new gen_ai.* attributes not previously seen (run quarterly)
dependencies
| where timestamp > ago(30d)
| mv-expand bagKey = bag_keys(customDimensions)
| where tostring(bagKey) startswith "gen_ai."
| summarize first_seen = min(timestamp) by tostring(bagKey)
| order by first_seen desc
```

### Collision resolution priority

If a developer attribute or org-contract attribute collides with a new OTel spec attribute:

1. **OTel spec wins** — the spec definition takes precedence
2. The org-contract attribute is **deprecated** and enters the breaking-change process
3. The SDK-emitted spec attribute is used going forward (no code required — it auto-emits)
4. Developer attributes in the developer namespace are **the team's problem to rename** — platform team provides guidance but teams own the migration

---

## 12. Org Contract — Full Definition (Authoritative)

This section is the **single source of truth** for the org contract. When this document and any other document disagree, this section takes precedence.

### Contract version: 1.1 (March 2026)

---

### Span names (org-mandated)

| Span name | Pattern | SDK | Who writes it | Deviation permitted? |
|---|---|---|---|---|
| Session root | `session` | Both | Developer | ❌ No |
| Session root (MAF) | `Scenario: {description}` | MAF only | Developer | ⚠️ Only the description part |
| Turn | `turn-{integer}` | Both | Developer | ❌ No — `turn-1` not `Turn 1`, `TURN_1`, etc. |
| Agent lifecycle | `agent.{action}` | `azure-ai-agents` | Developer | ⚠️ Action part is yours |
| Sub-operations | `snake_case verb phrase` | Both | Developer | ✅ Developer choice |

---

### Attributes injected automatically by `GlobalEnrichmentProcessor` (v1.0)

Set the corresponding environment variable. Do not call `set_attribute()` for these.

| Attribute | Type | Env var | Required? | Notes |
|---|---|---|---|---|
| `app.service.name` | string | `SERVICE_NAME` | ✅ Required | Appears as Cloud role in App Insights |
| `app.version` | string | `APP_VERSION` | ✅ Required | Format: semver `1.2.3` |
| `deployment.env` | string | `ENVIRONMENT` | ✅ Required | Values: `dev`, `staging`, `prod` only |
| `org.team` | string | `TEAM_NAME` | ✅ Required | Lowercase kebab-case |
| `org.cost_center` | string | `COST_CENTER` | ✅ Required | Must match finance system cost center ID |

---

### Attributes set manually on the session span (v1.0)

| Attribute | Type | Required? | Value constraints | PII? |
|---|---|---|---|---|
| `session.id` | string | ✅ Required | UUID v4 | No |
| `agent.id` | string | ✅ Required | Foundry agent ID (`asst_*`) | No |
| `thread.id` | string | ✅ Required | Foundry thread ID (`thread_*`) | No |
| `total.turns` | int | ✅ Required | ≥ 1; set at session end | No |
| `user.id` | string | ✅ Required | Must be hashed (SHA-256) in staging/prod | ⚠️ PII if unhashed |

---

### Attributes set manually on each turn span (v1.0)

| Attribute | Type | Required? | Value constraints | PII? |
|---|---|---|---|---|
| `turn.number` | int | ✅ Required | Sequential from 1 | No |
| `turn.latency_ms` | float | ✅ Required | Milliseconds, ≥ 0 | No |
| `run.status` | string | ✅ Required | `"completed"`, `"failed"`, `"cancelled"` | No |
| `response.length` | int | ✅ Required | Character count, ≥ 0 | No |
| `user.query` | string | ⚠️ Dev only | Raw user input — disabled in staging/prod | ⚠️ PII |

---

### Attributes set manually on the agent run span (v1.0)

| Attribute | Type | Required? | Value constraints |
|---|---|---|---|
| `run.id` | string | ✅ Required | Foundry run ID (`run_*`) |
| `thread.id` | string | ✅ Required | Same value as on turn span |
| `agent.id` | string | ✅ Required | Same value as on session span |

---

### Optional org-recognized attributes (v1.1)

These are not required but are recognized by shared dashboards if present. Use them when applicable.

| Attribute | Type | Applicable when | Notes |
|---|---|---|---|
| `experiment.variant` | string | Running an A/B test | Lowercase kebab-case variant name |
| `feature.flag` | string | Feature flag active | Flag name as registered in feature flag system |
| `org.product` | string | Multiple products per team | Product line (more granular than team) |
| `org.squad` | string | Squad-level attribution needed | Squad name within a team |
| `agent.reused` | bool | Agent reuse pattern used | `True` if agent ID was cached; `False` if newly created |

---

### Prohibited patterns (always blocked)

| Pattern | Reason |
|---|---|
| Any attribute starting with `gen_ai.*` | OTel spec reserved — SDK already emits these |
| Any attribute starting with `service.*`, `cloud.*`, `exception.*`, `telemetry.*` | OTel spec reserved |
| Span name containing a user ID, query text, or any dynamic unique string | High cardinality; use attributes for variable data |
| `user.id` set to a raw email, name, or other direct identifier in staging/prod | PII policy violation |
| `user.query` set in any non-dev environment | Content recording policy violation |
| `SimpleSpanProcessor` in any non-dev/CI environment | Blocks main thread; production performance violation |

---

## 13. Administrator Checklist

### When onboarding a new team

- [ ] Team has received the `DEVELOPER_TUTORIAL.md` and `ATTRIBUTE_REFERENCE.md`
- [ ] Team's repository has the CI/CD lint gate configured
- [ ] Team has set required env vars in their CI/CD pipeline (`SERVICE_NAME`, `TEAM_NAME`, `COST_CENTER`, `ENVIRONMENT`, `APP_VERSION`)
- [ ] Team's `tracing_config.py` is the current version from the shared package — not a copy with local modifications
- [ ] Team's smoke test verified: spans appear in Application Insights with all required org-contract attributes
- [ ] Team's sampling rate is appropriate for their expected session volume

### When reviewing a contract change RFC

- [ ] Does the proposed attribute name conflict with any Layer 1 (OTel spec) name? Check the [OTel registry](https://opentelemetry.io/docs/specs/semconv/).
- [ ] Does the proposed name follow the namespace model (no reserved prefix violations)?
- [ ] Is the attribute's cardinality bounded? What is the maximum number of unique values?
- [ ] Is there a PII assessment?
- [ ] Are at least two teams requesting this, or does it unlock an org-wide dashboard?
- [ ] Is the migration guide written for teams not yet using this attribute?
- [ ] Has the dual-emit period been planned (for breaking changes only)?

### Quarterly platform review

- [ ] Run the "new gen_ai.* attributes" KQL scan against Application Insights — any new SDK auto-attributes?
- [ ] Check Application Insights ingestion cost trends — any unexpected cardinality increases?
- [ ] Review sampling rate adequacy — are prod error traces being captured at target rate?
- [ ] Confirm `get_thread_run` suppression is working — polling spans should be < 5% of total span count
- [ ] Verify all teams are on the current contract version — query for missing required attributes
- [ ] Review any developer attributes appearing on ≥ 3 services — promotion candidates?
- [ ] Check CI/CD lint gate coverage — any new AI agent services added without the gate?

---

*Last updated: March 2026 · Contract version: 1.1  
Maintained by: Platform Engineering / AI Observability Team  
See also: [`ATTRIBUTE_REFERENCE.md`](ATTRIBUTE_REFERENCE.md) · [`DEVELOPER_TUTORIAL.md`](DEVELOPER_TUTORIAL.md) · [`TRACING_GUIDE.md`](TRACING_GUIDE.md)*
