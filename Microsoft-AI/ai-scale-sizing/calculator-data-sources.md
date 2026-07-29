# Calculator Data Sources & Method Notes

_Seeded July 8, 2026. Both calculators (`aoai-capacity-cost-calculator.html` and `.xlsx`) share this data and methodology. All values are editable in the tools — treat these seeds as a starting point and verify before customer commitments._

## Throughput / PTU metrics (authoritative)

Source: [Provisioned throughput unit (PTU) costs and billing](https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/provisioned-throughput-onboarding) (ms.date 2026-04-29) and [Provisioned throughput concepts](https://learn.microsoft.com/en-us/azure/foundry/openai/concepts/provisioned-throughput) (ms.date 2026-05-22).

| Model | Input TPM/PTU | Min PTU (Global/DZ) | Incr | Min PTU (Regional) | Incr |
|---|---|---|---|---|---|
| gpt-5.5 | 1,200 | 15 | 5 | 50 | 50 |
| gpt-5.4 | 2,400 | 15 | 5 | 50 | 50 |
| gpt-5.4-mini | 7,900 | 15 | 5 | 25 | 25 |
| gpt-5.3-codex / 5.2 / 5.2-codex | 3,400 | 15 | 5 | 50 | 50 |
| gpt-5.1 / 5.1-codex / gpt-5 | 4,750 | 15 | 5 | 50 | 50 |
| gpt-5-mini | 23,750 | 15 | 5 | 25 | 25 |
| gpt-4.1 | 3,000 | 15 | 5 | 50 | 50 |
| gpt-4.1-mini | 14,900 | 15 | 5 | 25 | 25 |
| gpt-4.1-nano | 59,400 | 15 | 5 | 25 | 25 |
| gpt-4o | 2,500 | 15 | 5 | 50 | 50 |
| gpt-4o-mini | 37,000 | 15 | 5 | 25 | 25 |
| o3 | 3,000 | 15 | 5 | 50 | 50 |
| o4-mini | 5,400 | 15 | 5 | 25 | 25 |

Key rules from the docs:

- **Output ratio**: one output token counts as N input tokens toward PTU utilization. For gpt-4.1+ models the ratio equals the model's Global Standard output/input price ratio (gpt-5 family = 8, gpt-4.1 family = 4, gpt-5.4/5.5 = 6 based on published prices). Older models "use a different ratio" per the docs — the tools default them to 4; verify with the Foundry capacity calculator.
- **Cache**: cached tokens are deducted 100% from PTU utilization, so normalized TPM = input TPM × (1 − cache rate) + output TPM × ratio.
- **PTUs required** = peak normalized TPM ÷ input TPM per PTU, rounded up to the min/increment for the deployment type.

## PAYGO pricing (Global Standard, $ per 1M tokens)

Sources: Azure OpenAI pricing page and aggregators (confirmed via search July 2026). gpt-5.5 $5.00/$30.00; gpt-5.4 $2.50/$15.00; gpt-5.2 $1.75/$0.175 cached/$14.00; gpt-5 $1.25/$0.125/$10.00; gpt-5-mini $0.25/$2.00; gpt-4.1 $2.00/$0.50/$8.00; gpt-4.1-mini $0.40/$1.60; gpt-4.1-nano $0.10/$0.40; gpt-4o $2.50/$10.00; gpt-4o-mini $0.15/$0.60; o3 $2.00/$8.00; o4-mini $1.10/$4.40.

**Marked as estimates in the tools** (couldn't confirm from a primary source): gpt-5.4-mini, gpt-5.1. Verify all against the [Azure pricing calculator](https://azure.microsoft.com/pricing/details/azure-openai/).

Deployment-type uplift: modeled as a multiplier on Global Standard (defaults: Data Zone ×1.10, Regional ×1.20). Microsoft applies a 10% uplift on regional data-residency endpoints for models released on/after Mar 5, 2026; older models vary — adjust per region.

## PTU rates ($ per PTU)

Defaults (verify — these are the long-standing published rates, confirmed only partially in current sources):

| Deployment | Hourly | Monthly resv | Yearly resv |
|---|---|---|---|
| Global | $1.00/hr | $260 | $2,652 |
| Data Zone | $1.10/hr | $286 | $2,916.60 |
| Regional | $2.00/hr | $520 | $5,304 |

The $260/mo and $2,652/yr Global reservation rates were confirmed in current sources; Data Zone and Regional carried forward from prior published rates. There is no public PTU pricing API — see `foundry-rest-api.md` in this folder for the API landscape (Retail Prices API covers PAYGO meters only).

## Calculation conventions

- Weeks/month = 365.25 ÷ 7 ÷ 12 ≈ 4.348 (matches the legacy sizing artifacts: 50 h/wk → 217.41 h/mo).
- Hours/month = 730.5.
- PAYGO $/mo = Σ periods [fresh input × input rate + cached input × cached rate + output × output rate] × deployment multiplier ÷ 1M.
- PTU groups: use cases sharing model + PTU deployment type share one deployment; sizing sums normalized TPM per load period and takes the peak (assumes coincident peaks — worst case).
- Utilization = normalized token-minutes used ÷ deployed capacity over the month.
- Yearly reservation shown amortized per month (÷12).

## Validation

Both tools were cross-checked against each other and against the legacy artifacts in `legacy-sizing-artifacts/` (hours/month and monthly token volumes reproduce exactly; costs differ only due to updated model prices). Final sizing should still be validated with the [Foundry capacity calculator](https://ai.azure.com/resource/calculator) — real throughput varies with call-shape distribution and burstiness.
