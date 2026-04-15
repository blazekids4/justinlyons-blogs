# Latency-Safe Scaling Strategy for Azure OpenAI Using PTUs

## Objective

Provide **predictable, low-latency performance** for business-critical Azure OpenAI workloads while retaining flexibility to handle demand variability and future growth.

This approach uses a **three-tier defense-in-depth model for capacity**, where each layer has a clearly defined role. Latency and reliability come first, with disciplined cost and capacity management over time.

> *"Size Data Zone PTUs for everything you expect, Global PTUs for controlled surge, and PAYG only for truly unexpected events. PAYG tells you when reality has changed — then resize PTUs accordingly."*

---

## Core Principles

1. **PTUs are used for all latency-sensitive traffic.**
   Provisioned Throughput Units provide reserved, predictable model capacity and are the only mechanism that guarantees consistent latency under load.

2. **Known peaks are provisioned, not auto-scaled.**
   PTUs do not auto-scale. Any predictable peak (e.g., morning spikes) must be handled through intentional PTU buffering.

3. **PAYG is a safety valve, not a scaling mechanism.**
   Pay-as-you-go (token-based) deployments are reserved for unexpected bursts, anomalous traffic, and early signals that baseline demand has increased.

---

## Three-Tier Deployment Model

### Tier 1 — Data Zone PTUs (Baseline Capacity)

The primary line of defense. Carries all expected traffic.

- Sized for steady-state load **and** known, recurring peak hours
- Ensures data residency compliance, predictable throughput, and consistent latency
- Highest confidence — this layer should handle *everything you expect to happen*

**Example:** 300 PTUs reserved in **Data Zone US** to cover baseline and expected peak demand.

---

### Tier 2 — Global PTUs (Surge Capacity)

Planned elasticity without relaxing latency requirements.

- Invoked when Data Zone PTUs are saturated or demand exceeds the expected peak envelope
- Still PTU-backed, so latency remains stable
- Broader backend capacity pools and faster access to incremental capacity

**Example:** 100 PTUs provisioned in **Global** for controlled surge handling.

This avoids over-inflating Data Zone PTUs while maintaining latency guarantees.

---

### Tier 3 — PAYG (Burst & Anomaly Protection)

Last resort. Used only when **both** PTU tiers are exceeded — which should be rare.

- Handles unexpected spikes, traffic anomalies, and growth that outpaces forecast
- No capacity reservation; latency variability is possible
- Monitored closely, alerted on, and optionally rate-limited to manage cost exposure

PAYG is not there to scale the system. It is there to **prevent failure while you learn**.

---

## How Traffic Flows

```
Normal load
   ↓
Data Zone PTUs

Data Zone saturated
   ↓
Global PTUs

Global saturated (unexpected)
   ↓
PAYG (alerts fire, investigation begins)
```

---

## Observability & Feedback Loop

The environment is continuously monitored to answer three key questions:

1. **How often is Global capacity invoked?**
   Indicates whether Data Zone PTUs are consistently saturated.

2. **How deep and frequent are surge events?**
   Measures true peak TPM and duration, not one-off spikes.

3. **Are new peak patterns sustained over time?**
   Distinguishes transient anomalies from structural demand growth.

---

## PTU Right-Sizing Strategy

| Signal | Action |
|---|---|
| Global PTUs used **occasionally** | No action required |
| Global PTUs engaged **frequently and predictably** | Baseline is undersized — increase Data Zone PTUs |
| PAYG usage meaningfully increases | Investigate upstream demand changes |

When sustained trends are observed, baseline capacity is adjusted:

1. Increase **Data Zone PTUs**
2. Reduce reliance on Global surge capacity
3. Maintain PAYG as a safety net

This creates an intentional, data-driven resize loop rather than speculative provisioning.

---

## Capacity & Risk Considerations

- PTU allocations are subject to **regional and model availability**
- Data Zone capacity is typically more constrained than Global
- Short-notice PTU increases cannot be assumed to succeed
- This strategy explicitly plans for those realities

---

## Why This Model Works

- **Latency is protected first** — PAYG never becomes the primary scaling mechanism
- **Capacity decisions are data-driven** — PAYG usage tells you something changed, not that the system is broken
- **Financial exposure is controlled** — PAYG is bounded, visible, and intentional
- **Resizing is deliberate, not reactive** — sustained Global/PAYG usage triggers a Data Zone PTU resize

This is a conservative, production-proven approach designed for enterprise-scale AI workloads where latency and reliability are non-negotiable.
