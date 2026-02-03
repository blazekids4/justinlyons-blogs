# Azure OpenAI Scale & Cost Sizing Workshop

## Required Metrics Collection Guide

### Workshop Approach

**Our Recommendation**: We'll start by sizing your most common use case patterns and extrapolate those across your broader application portfolio. This inference-first approach will establish a solid baseline for capacity planning, which we can then use to price the ancillary Azure services (storage, compute, networking, etc.) that support your AI applications.

**Why This Works**:

- Once we understand your token consumption patterns, ancillary service sizing becomes more predictable
- Allows us to identify optimization opportunities early in the design phase

---

## Phase 1: Common Use Case Baseline

### Step 1: Identify Your Core Patterns

Please categorize your AI workloads into **3-5 common use case archetypes**

For example:

| Archetype Example         | Characteristics                                       | Instances                            |
| ------------------------- | ----------------------------------------------------- | ------------------------------------ |
| **Interactive Assistant** | Real-time chat, moderate context, streaming responses | Customer support, IT helpdesk        |
| **Document Processing**   | Large context windows, batch operations, async        | Contract analysis, compliance review |
| **Code Generation**       | Medium context, high token generation, iterative      | Developer copilot, SQL generation    |
| **RAG Q&A**               | Vector search + LLM, knowledge retrieval              | FAQ bot, knowledge base              |
| **Agent Workflow**        | Multi-step, function calling, extended sessions       | Research agent, data analyst         |

**Action Item**: List your use cases and map them to these or similar patterns.

---

## Phase 2: Metrics Collection Per Archetype

For **each common use case pattern**, provide the following:

### 1. User Activity Metrics

| Metric                   | Description                                                 | Example                |
| ------------------------ | ----------------------------------------------------------- | ---------------------- |
| **Total Users**          | Number of users with access to solutions using this pattern | 5,000                  |
| **Active User %**        | Percentage typically active during peak periods             | 25% (1,250 concurrent) |
| **Sessions per User**    | Average sessions per user per day/period                    | 3 sessions/day         |
| **Usage Timeframe**      | Primary hours of operation                                  | 8am-5pm ET, Mon-Fri    |
| **Requests per Session** | Average API calls per user session                          | 15 requests            |

**RPM Calculation**:

```
(Total Users × Active % × Sessions × Requests) / Usage Window Minutes = Peak RPM
```

---

### 2. Token Consumption Profile

| Metric                     | Description                                  | Example                          |
| -------------------------- | -------------------------------------------- | -------------------------------- |
| **System Prompt Length**   | Token count of your system instructions      | 500 tokens                       |
| **Function Definitions**   | Number of tools/functions available to model | 8 functions (~50 tokens each)    |
| **Conversation Length**    | Average number of turns per conversation     | 6 turns                          |
| **User Message Size**      | Typical tokens per user message              | 150 tokens                       |
| **Model Response Size**    | Expected tokens per model response           | 300 tokens                       |
| **Few-Shot Examples**      | Number and size of examples in prompt        | 3 examples × 200 tokens          |
| **RAG Context Chunks**     | Number and size of retrieved documents       | 5 chunks × 400 tokens            |
| **Images** (if applicable) | Images per request + resolution              | 2 images, 1024×1024, high detail |

**Token Calculation**:

```
Context Tokens = System Prompt + Functions + (Conversation Turns × Msg Size) + Few-Shot + RAG
Generation Tokens = Response Size
```

---

### 3. Load Pattern Distribution

Define when this pattern gets used:

| Period Name    | Days    | Start Time | End Time | % of Daily Volume |
| -------------- | ------- | ---------- | -------- | ----------------- |
| Business Hours | Mon-Fri | 08:00      | 17:00    | 70%               |
| After Hours    | Mon-Fri | 17:00      | 08:00    | 20%               |
| Weekend        | Sat-Sun | 00:00      | 24:00    | 10%               |

---

### 4. Model & Deployment Preferences

| Field                    | Selection                        | Notes                              |
| ------------------------ | -------------------------------- | ---------------------------------- |
| **Model Family**         | GPT-4o / o1 / GPT-4o-mini        | Based on quality/cost requirements |
| **Model Version**        | Latest stable / Specific version | For reproducibility                |
| **PTU Deployment Type**  | Global / Data Zone / Regional    | Availability vs latency tradeoff   |
| **Purchase Commitment**  | Monthly / Yearly / Hourly        | Cost optimization strategy         |
| **Negotiated Discounts** | % (if applicable)                | EA/MACC/Partner agreements         |

---

## Phase 3: Extrapolation Framework

### Use Case Inventory Template

Once we baseline your common patterns, map all remaining use cases:

| Use Case Name        | Maps to Archetype     | Monthly Active Users | Multiplier        | Notes                    |
| -------------------- | --------------------- | -------------------- | ----------------- | ------------------------ |
| Customer Support Bot | Interactive Assistant | 5,000                | 1.0×              | Baseline pattern         |
| Sales Assistant      | Interactive Assistant | 2,500                | 0.5×              | Half the volume          |
| Contract Review      | Document Processing   | 150                  | See separate calc | Batch, different profile |
| HR Knowledge Base    | RAG Q&A               | 8,000                | 1.2×              | Slightly higher usage    |

**Extrapolation Logic**:

- Use cases sharing an archetype inherit the token profile
- Scale RPM based on user count and activity multipliers
- Adjust for variance (e.g., seasonal spikes, departmental differences)

---

## Workshop Deliverables

### Primary Outputs (Inference Baseline)

1. **Capacity Requirements Per Archetype**
   - Peak TPM (Tokens Per Minute)
   - Required PTUs (Provisioned Throughput Units)
   - Total RPM capacity needed

2. **Cost Forecast**
   - Monthly token volume by use case pattern
   - Pay-As-You-Go baseline costs
   - PTU costs (with optimization scenarios)
   - Projected savings (PTU vs PayGo)

3. **Extrapolated Portfolio View**
   - Total organizational inference spend
   - Growth projections based on planned rollouts
   - Cost per user/session benchmarks

### Secondary Outputs (Ancillary Services Sizing)

Once inference baseline is established, we'll size:

4. **Dependent Azure Services**
   - **Azure AI Search**: Index size, queries/sec, tier recommendations
   - **Azure Storage**: Document volumes, blob/file storage needs
   - **Azure Functions/Container Apps**: Orchestration compute requirements
   - **Azure Monitor/App Insights**: Observability and logging costs
   - **Networking**: VNet, Private Link, egress estimates
   - **others**

5. **Total Solution TCO**
   - Inference (largest component)
   - - Data & vector storage
   - - Compute orchestration
   - - Monitoring & governance
   - = Complete monthly run rate

---

## Estimation Guidance

### If You Don't Have Exact Metrics

**Token Estimates**:

- Use [OpenAI Tokenizer](https://platform.openai.com/tokenizer) for prompt samples
- Rule of thumb: 1 token ≈ 4 characters for English text
- Test with representative documents/conversations

**User Activity**:

- Start with application analytics (sessions, page views)
- Use proxy metrics (support tickets, CRM interactions)
- Estimate conservative, plan for 2-3× growth

**Load Patterns**:

- Default to business hours (9am-5pm) for interactive patterns
- Assume 80/20 distribution (80% peak, 20% off-peak)
- Refine post-pilot with actual telemetry

---

## Pre-Workshop Checklist

- [ ] Identify 3-5 common use case archetypes
- [ ] Complete metrics for each archetype (tables above)
- [ ] Map remaining use cases to archetypes
- [ ] Gather model preferences and deployment constraints
- [ ] Document any existing Azure commitments (EA, MACC)
- [ ] Identify stakeholders for ancillary service discussions

---

## FAQ

**Q: What if we have dozens of use cases?**  
A: That's why we archetype first. You likely have 3-5 core patterns that account for 80%+ of consumption. We size those deeply, then extrapolate.

**Q: Should we size for current state or future state?**  
A: Start with 6-month forward view. We can always adjust as you onboard more workloads.

**Q: What about dev/test environments?**  
A: We'll account for non-production in the workshop. Typically some % of production capacity.

---

## Appendix: The Four Core Drivers

Everything rolls up to these inputs:

1. **User Behavior** → RPM per use case
2. **Prompt Design** → Tokens per request (context + generation)
3. **Load Distribution** → When peak usage occurs
4. **Model Selection** → Pricing tier and throughput characteristics

**Master Formula**:

```
Monthly Cost = Σ (Use Case RPM × Tokens/Request × Active Hours × Model Price)
```

Archetype-based sizing lets us solve this once per pattern, not once per use case.

---

_Document Version: 1.0_  
_Last Updated: January 2026_
_Prepared by: Justin Lyons, Solutions Engineer - AI Apps & Agents, Microsoft_
