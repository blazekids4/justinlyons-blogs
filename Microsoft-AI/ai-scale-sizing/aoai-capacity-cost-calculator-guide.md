# Azure OpenAI Capacity and Cost Calculator: How-To Guide

Use the [Azure OpenAI Capacity and Cost Calculator](aoai-capacity-cost-calculator.html) to estimate Azure OpenAI token consumption, Pay-As-You-Go (PayGO) cost, and Provisioned Throughput Unit (PTU) requirements by use case.

> [!IMPORTANT]
> The calculator does not automatically retrieve current Azure model, throughput, or pricing data. Its built-in values are editable seed values, not a live price list. Before every sizing exercise, manually update and validate the model catalog, PTU throughput values, deployment minimums and increments, PayGO prices, deployment multipliers, and PTU rates. Models, availability, throughput, and prices fluctuate by date, region, deployment type, and commercial agreement.

The calculator is a planning aid. Do not use its output as a quote, capacity guarantee, or purchasing commitment.

## Open the calculator

1. Download or clone this repository.
2. Open `aoai-capacity-cost-calculator.html` in a modern web browser.
3. Use **Reset to defaults** only when you intentionally want to discard all saved inputs.

The calculator runs entirely in the browser. It does not require a web server, send data to Azure, or call an Azure pricing API. Changes are saved in the browser's local storage.

## Required data refresh

Complete this step before entering workload data.

1. Open the **Models & Pricing** tab.
2. Remove obsolete models and add any models required by the workload.
3. For every model, verify and update:
   - Input TPM per PTU
   - Output token ratio
   - Global or Data Zone minimum PTUs and increments
   - Regional minimum PTUs and increments
   - Input price per 1 million tokens
   - Cached input price per 1 million tokens
   - Output price per 1 million tokens
4. Verify the hourly, monthly reservation, and yearly reservation PTU rates for each deployment type.
5. Verify the PayGO multiplier for Global, Data Zone, and Regional deployments.
6. Confirm that the selected model is available in the target Azure region and deployment type.

Use current Microsoft sources, including:

- [Azure OpenAI Service pricing](https://azure.microsoft.com/pricing/details/azure-openai/)
- [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/)
- [Provisioned throughput concepts](https://learn.microsoft.com/azure/ai-services/openai/concepts/provisioned-throughput)
- [Provisioned throughput onboarding and billing](https://learn.microsoft.com/azure/ai-services/openai/how-to/provisioned-throughput-onboarding)
- [Microsoft Foundry capacity calculator](https://ai.azure.com/resource/calculator)

See [Calculator Data Sources & Method Notes](calculator-data-sources.md) for the origin and limitations of the seeded values.

> [!CAUTION]
> Rows marked `est.` contain estimated values. Unmarked rows must still be checked. A value that was accurate when the file was created may no longer match the target model version, region, deployment type, or current Azure price.

## Define load periods

Open the **Load Periods** tab and divide the month into periods with different traffic levels. The defaults separate weekday business hours, weekday off-peak hours, and weekends.

For each period, enter:

- A descriptive name
- Hours per day
- Days per week

The calculator converts these values to monthly hours using approximately 4.348 weeks per month. For a full-month forecast, the periods should total approximately 730.5 hours per month. A different total can be valid when intentionally modeling only part of the month, but gaps and overlaps should be documented.

## Add use cases

Open the **Use Cases** tab. Add one row for each workload pattern that has meaningfully different traffic, token usage, model selection, or caching behavior.

For each use case, enter:

| Field                 | What to enter                                                 |
| --------------------- | ------------------------------------------------------------- |
| Use case              | A recognizable workload name                                  |
| Model                 | The model configured in the refreshed catalog                 |
| PayGO deployment      | The deployment type used to price PayGO tokens                |
| PTU deployment        | The provisioned deployment pool the use case would join       |
| Input tokens/request  | Average input tokens for one request                          |
| Output tokens/request | Average generated tokens for one request                      |
| Cache %               | Expected percentage of input tokens served from prompt cache  |
| RPM by period         | Average sustained requests per minute during each load period |

Use representative production telemetry where available. Averages that hide large request shapes or short traffic spikes can understate required capacity.

The **RPM helper** can estimate RPM from users, activity, sessions, requests per session, and the daily usage window. Treat its result as a starting point and replace it with measured or load-tested demand when possible.

Use the duplicate action when two workloads share a similar starting profile, then adjust the copied row. Use cases with the same model and PTU deployment type are grouped into one potential PTU deployment.

## Review results

Open the **Results** tab after the inputs are complete.

### Aggregate summary

The summary compares:

- **100% PayGO/month**: calculated token charges across all use cases and load periods
- **100% PTU/month**: cost of the rounded PTU capacity under the selected billing basis
- **Optimized mix/month**: the lower calculated option for each model and deployment group
- **PTU vs PayGO**: the percentage difference between the two strategies

Select the applicable PTU billing basis: hourly, monthly reservation, or yearly reservation amortized monthly.

### PTU deployment groups

The calculator groups use cases by model and PTU deployment type because one provisioned deployment serves one model. For each group, review:

- Peak raw and normalized TPM
- Exact calculated PTUs
- PTUs rounded to the configured minimum and increment
- Estimated utilization
- PayGO and PTU monthly costs
- The lower-cost option under the selected assumptions

Group sizing assumes peaks occur at the same time within a load period. This is conservative when workloads peak at different times, but broad periods can also conceal short bursts.

### Per-use-case and period detail

Use the per-use-case table to identify the workloads driving token volume, peak throughput, and PayGO cost. Expand each period detail to verify RPM, monthly hours, token volume, normalized TPM, cached tokens, and cost.

## Understand the calculations

For each load period:

$$
\text{Input TPM} = \text{RPM} \times \text{input tokens per request}
$$

$$
\text{Output TPM} = \text{RPM} \times \text{output tokens per request}
$$

The PayGO estimate is:

$$
\text{PayGO cost} = \frac{(\text{fresh input} \times \text{input price}) + (\text{cached input} \times \text{cached price}) + (\text{output} \times \text{output price})}{1{,}000{,}000}
$$

PTU sizing uses normalized TPM:

$$
\text{Normalized TPM} = \text{input TPM} \times (1 - \text{cache rate}) + \text{output TPM} \times \text{output ratio}
$$

$$
\text{Exact PTUs} = \frac{\text{peak normalized TPM}}{\text{input TPM per PTU}}
$$

Exact PTUs are rounded up to the configured minimum and increment for the deployment type. Actual throughput can differ because of model version, request shape, output generation, burstiness, latency targets, and service-side behavior.

## Save or share a scenario

Select **Export JSON** to download the complete calculator state, including model assumptions, pricing, periods, use cases, and billing selection. Select **Import JSON** to restore a previously exported scenario.

Include the export date and assumption source dates when sharing a scenario. Imported files also contain point-in-time values and must be refreshed before reuse.

## Final validation checklist

Before presenting or acting on the estimate, confirm that:

- Every required model and version is currently available in the target region and deployment type.
- All model throughput values, output ratios, minimums, and increments were manually refreshed.
- All PayGO token prices, cache prices, multipliers, and PTU rates were manually refreshed.
- Currency, negotiated discounts, reservation terms, and taxes are handled outside the calculator as needed.
- RPM, token sizes, cache rates, and load periods are supported by telemetry, testing, or documented assumptions.
- Peak traffic, concurrency, quota, latency, failover, and growth headroom have been considered.
- PTU estimates were checked in the Microsoft Foundry capacity calculator with representative call shapes.
- Final prices were checked in the Azure pricing calculator or confirmed through the appropriate Microsoft or account team channel.

Record the validation date and retain the exported JSON with the estimate. Revalidate whenever the model, deployment region, workload shape, reservation term, or pricing changes.
