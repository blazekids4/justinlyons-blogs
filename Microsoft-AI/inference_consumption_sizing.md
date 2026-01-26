# Azure OpenAI Scale & Cost Sizing Workshop

## Required Metrics Collection Guide

### Overview

This document outlines the metrics required to accurately forecast Azure OpenAI capacity needs and costs. Please gather this information for each AI use case prior to the workshop.

---

## 1. User Activity Metrics

| Metric                   | Description                                     | Example                |
| ------------------------ | ----------------------------------------------- | ---------------------- |
| **Total Users**          | Number of users with access to the solution     | 5,000                  |
| **Active User %**        | Percentage typically active during peak periods | 25% (1,250 concurrent) |
| **Sessions per User**    | Average sessions per user per day/period        | 3 sessions/day         |
| **Usage Timeframe**      | Primary hours of operation                      | 8am-5pm ET, Mon-Fri    |
| **Requests per Session** | Average API calls per user session              | 15 requests            |

---

## 2. Token Consumption Metrics

| Metric                     | Description                                  | Example                          |
| -------------------------- | -------------------------------------------- | -------------------------------- |
| **System Prompt Length**   | Token count of your system instructions      | 500 tokens                       |
| **Function Definitions**   | Number of tools/functions available to model | 8 functions                      |
| **Conversation Length**    | Average number of turns per conversation     | 6 turns                          |
| **User Message Size**      | Typical tokens per user message              | 150 tokens                       |
| **Model Response Size**    | Expected tokens per model response           | 300 tokens                       |
| **Few-Shot Examples**      | Number and size of examples in prompt        | 3 examples × 200 tokens          |
| **RAG Context Chunks**     | Number and size of retrieved documents       | 5 chunks × 400 tokens            |
| **Images** (if applicable) | Images per request + resolution              | 2 images, 1024×1024, high detail |

### Outcome Calculations

- **Avg Context Tokens/Request** = System prompt + Functions + Conversation history + Few-shot + RAG chunks + Images
- **Avg Generation Tokens/Request** = Model response size

---

## 3. Load Pattern Details

### Load Periods

Define distinct usage patterns across your workday/week:

| Period Name    | Days    | Start Time | End Time | Description          |
| -------------- | ------- | ---------- | -------- | -------------------- |
| Business Hours | Mon-Fri | 08:00      | 17:00    | Peak agent usage     |
| After Hours    | Mon-Fri | 17:00      | 08:00    | Automated processing |
| Weekend        | Sat-Sun | 00:00      | 24:00    | Minimal usage        |

### Use Case Configuration (per load period)

| Field                     | Description                            | Example                |
| ------------------------- | -------------------------------------- | ---------------------- |
| **Use Case Name**         | Descriptive identifier                 | Customer Support Agent |
| **Model & Version**       | Azure OpenAI model selection           | GPT-4o (2024-11-20)    |
| **Avg Context Tokens**    | From calculation above                 | 3,200 tokens           |
| **Avg Generation Tokens** | From calculation above                 | 300 tokens             |
| **RPM per Period**        | Requests per minute during this period | 450 RPM                |

---

## 4. Deployment & Pricing Preferences

| Preference                | Options                                                             | Notes                              |
| ------------------------- | ------------------------------------------------------------------- | ---------------------------------- |
| **PTU Deployment Type**   | • Global Standard<br>• Data Zone Standard<br>• Regional             | Global offers highest availability |
| **PayGo Deployment Type** | • Global Standard<br>• Data Zone Standard<br>• Regional             | For comparison baseline            |
| **PTU Purchase Option**   | • Monthly commitment<br>• Yearly commitment<br>• Hourly (on-demand) | Yearly offers best pricing         |
| **Negotiated Discounts**  | % discount (if applicable)                                          | EA/MACC discounts                  |

---

## Workshop Outputs

Using your metrics, we will calculate:

1. **Capacity Requirements**
   - Peak Tokens Per Minute (TPM)
   - Provisioned Throughput Units (PTUs) needed
   - Requests Per Minute (RPM) capacity

2. **Cost Analysis**
   - Monthly token volume projection
   - Pay-As-You-Go estimated costs
   - PTU monthly costs (peak + average scenarios)
   - Cost savings percentage (PTU vs PayGo)

3. **Deployment Recommendations**
   - Optimal deployment type for your workload
   - Scaling strategy
   - Cost optimization opportunities

---

## How to Estimate Unknown Metrics

If you don't have exact measurements:

- **Token counts**: Use [OpenAI Tokenizer](https://platform.openai.com/tokenizer) to estimate prompt/response sizes
- **User activity**: Review application logs or analytics for session/request patterns
- **Load patterns**: Start with business hours assumption, refine with actual usage data
- **Conversation length**: Estimate based on your UX design (e.g., 3-turn Q&A vs 10-turn dialogue)

---

## Appendix: Quick Reference

### The 4 Core Inputs That Drive Everything

1. **User Behavior** → Requests Per Minute (RPM)
2. **Prompt Design** → Tokens per request (context + generation)
3. **Load Periods** → When usage happens (peak vs off-peak)
4. **Model Choice** → Pricing tier and throughput requirements

These four inputs determine your entire capacity and cost forecast.

---

_Document Version: 1.0_  
_Last Updated: January 2025_
