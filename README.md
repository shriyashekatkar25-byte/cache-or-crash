# cache-or-crash

**How big should the default per-session cache budget be when DataFrame caching on serverless compute goes from Preview to everyone?** A Monte Carlo integration + importance sampling decision prototype.

- **Live demo:** https://shriyashekatkar25-byte.github.io/cache-or-crash/
- **Gallery:** [APM Lab](https://shriyashekatkar25-byte.github.io/apm-lab/), product-decision prototypes by Shriya Shekatkar (UCLA, Statistics & Data Science minor · Stats 102C Monte Carlo Methods)

## The launch that inspired it

On **October 23, 2026**, the Databricks platform release notes announced that *DataFrame caching on serverless compute is now in Public Preview*: `df.cache()` / `df.persist()` can reuse results within a session. It requires serverless environment version 5 or later, and admins can turn it off from the Previews page.
Sources: [Databricks platform release notes, October 2026](https://docs.databricks.com/aws/en/release-notes/product/2026/october) · [Serverless compute limitations: caching](https://docs.databricks.com/aws/compute/serverless/limitations)

## The PM decision

Imagine I'm the PM taking this feature to GA. The hard call is the **default per-session cache budget**. Too small, and sessions keep evicting, so users pay to materialize caches they never get to read. Too large, and reserved memory gets expensive. Neither shows the real risk, which is the rare session whose cache demand blows past the budget and crashes. That tail is too rare for plain sampling to see, so I estimate it with importance sampling.

| | |
|---|---|
| Customer | Data scientists iterating in serverless notebooks, reusing one expensive DataFrame many times |
| Change | Default per-session cache budget (2–64 GB), LRU eviction, session restart beyond 3× budget |
| Success metric | ≥ 2.0 compute minutes saved per caching session; ≥ 90% of sessions fully in budget |
| Guardrails | Crashes ≤ 1 per 10,000 sessions · sessions made slower ≤ 2% · reserved-memory cost ≤ 25% of compute $ saved · p95 latency ±5% |
| Stakeholders | Serverless compute eng, notebook/ML experience PMs, Finance/COGS, Support & docs, field engineers |

With the default synthetic assumptions, the scan picks **16 GB**. At 8 GB the crash tail is about 10 per 10k, which fails the guardrail. At 64 GB, reserved memory costs too much. Move the sliders and the director memo rewrites itself.

## What this shows about product judgment

- **Customer problem framing:** starts from the iterative-notebook user and the waiting cost of recomputing a lineage.
- **Prototyping:** a working, interactive simulation instead of a slide.
- **SQL / Python:** an illustrative query to fit size and reuse distributions from Preview telemetry, plus a vectorized NumPy importance sampler.
- **Data-driven decisions:** every candidate budget is scored on one success metric and four guardrails. The pick is the smallest budget that passes all of them.
- **Clear writing:** a director memo (decision, evidence, risks, rollout, next step) and a one-sentence VP summary.

## Methods (Stats 102C)

| Method | Where it's used |
|---|---|
| Monte Carlo integration (Ch6) | Expected minutes saved per session as a sample mean, with a running ±1.96·SE band |
| Importance sampling (Ch6) | Crash probability: log-sizes drawn from a proposal shifted by δ, weights w = Π f/g; reports mean weight, SE and efficiency vs plain MC |
| Inverse CDF (Ch3) | Geometric reuse counts, R = 1 + ⌊ln U / ln(1−p)⌋ |
| Box-Muller (Ch5) | Normal draws for log-normal DataFrame sizes |
| Seeded RNG (Ch1) | mulberry32 seed, so every run is reproducible; common random numbers across budgets |

## Design baseline

Four pins from Shriya's Pinterest home feed, each redrawn in CSS/SVG. No pin images are used, embedded or traced.

- **Hero:** [“programming rule” by Ashish Kumar](https://www.pinterest.com/pin/704391198011909232/). Huge condensed grotesk, fully justified black-on-white lines with wide word gaps and a soft blur.
- **Cards & buttons:** [“Where flowers dance” by Tinyghetto](https://www.pinterest.com/pin/1109785533256726799/). Graph-paper grid, black ink flash ornaments (quatrefoil flowers), heavy outlined frames.
- **KPI badges:** [elephants pin (untitled, October 2026 board)](https://www.pinterest.com/pin/316026098843747403/). Flat cyan field (#1fb3e5), chunky grey shapes with thick black outlines, little pink “toe” dots.
- **Simulation room:** [“Shree Radha Krishna in Chinese attire” by Star Boy](https://www.pinterest.com/pin/875246508858164242/). Sage-green ground (#8f9e86), dark brown frame (#4b3415), saffron gold (#d9a62b) halo, deep teal (#1f5563), rose (#d98c99), brocade-style pattern and a serif label (palette and framing only).

---

Unofficial concept prototype by Shriya Shekatkar. All data is simulated. Not affiliated with any company.
