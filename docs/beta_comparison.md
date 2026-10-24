# Attachment exponent β: first comparison (2026-10-24)

What changes when the preferential attachment exponent is reduced from the SASCA-ReSA default β_P = 3 to the linear β_P = 1?

## Setup
- **Input:** multifield seed network (26,686 nodes, 273,490 edges); 30 simulated years at 13.42% annual growth, ending at about 1.17M nodes and 21.9M edges.
- **Baseline:** α random per agent, phenotype weights random per agent, no planted nodes, no cartels. Sample config: [`SASCA-ReSA-INSTAR.sample.config.ini`](SASCA-ReSA-INSTAR.sample.config.ini).
- **Conditions:**

| Condition | Runs | Build |
|---|---|---|
| β = 3 (all four scores) | 5 replicates | SASCA-ReSA 6.2.8 |
| β = 3 (all four scores) | 1 | this repo, 6.3.0, defaults |
| β_P = 1 (`pa_exponent=1`; fitness/authors/reputation stay 3) | 3 replicates | this repo, 6.3.0 |
| β = 3, α = 0.9 | 2 replicates | SASCA-ReSA 6.2.8 |
| β = 3, α = 0.1 | 2 replicates | SASCA-ReSA 6.2.8 |

## Measures
- **γ**: tail exponent of the in-degree distribution, N(k) ∼ k^(−γ). Discrete power-law MLE with k_min chosen by minimum KS distance (Clauset, Shalizi & Newman 2009).
- **β_eff**: effective attachment kernel of the grown network (Jeong, Néda & Barabási 2003). For each simulated year: nodes binned by in-degree k at the start of the year; citations received during the year per node, normalized by that year's mean. Computed within node-age bands, because recency otherwise confounds k with age. β_eff is the weighted log–log slope for k ≥ 20 over bins with ≥ 100 node-years. In practice this covers k up to about 250–380.
- **Concentration**: largest in-degree, and the share of all citations received by the top 1% most-cited nodes.

## Results

| Condition | γ | β_eff by age (yr): 3–5 / 6–10 / 11–20 / >20 | Max in-degree | Top-1% share |
|---|---|---|---|---|
| β = 3 (6.2.8) | 2.32–2.34 | 1.12 / 1.08 / 1.11 / 0.86 | 126k–230k | 0.42–0.43 |
| β = 3 (6.3.0) | 2.33 | 1.12 / 1.08 / 1.12 / 0.88 | 178k | 0.42 |
| **β_P = 1** | 2.35 | 1.11 / 1.08 / 1.09 / 0.85 | **97k–107k** | **0.35** |
| α = 0.9 | 2.70 | 1.20 / 0.94 / 1.07 / 0.92 | 44k–53k | 0.24 |
| α = 0.1 | 2.04 | 1.10 / 1.11 / 1.11 / 0.94 | 126k–134k | 0.54 |

Ranges are across replicates. Single values are means, except for single runs. The standard errors of individual γ fits (0.003–0.005) understate the uncertainty; the spread between replicates (about 0.02) is the better guide.

## Observations
1. **6.3.0 with defaults reproduces 6.2.8.** It falls inside the 6.2.8 replicate range on every measure.
2. **The effective kernel is about linear whatever β is.** With the agent-level rule either cubic (β = 3) or linear (β = 1), β_eff ≈ 1.1 for nodes aged 3–20 years, and γ barely changes. Candidates come from sampled neighborhoods and recency bins, are cited without replacement, and are scored by a mixture of four terms. Together these damp the agent-level exponent.
3. **β matters at the very top, beyond the range the kernel can be fitted on.** With β_P = 3, the largest hubs are bigger and far more variable: a 1.8× spread over 5 replicates, against 1.1× over 3 replicates with β_P = 1. The top 1% also take a larger share of citations. A superlinear rule amplifies chance among the most-cited nodes.
4. **Locality α is a much stronger lever than β_P** on both γ and concentration.
5. **The simulated tails are heavier than real ones.** γ ≈ 2.0–2.7 here, against Redner's (1998) γ ≈ 3 for real citation networks.

## Limitations
- One seed network.
- Few replicates for the β_P = 1 and α conditions.
- β was varied for the in-degree score only.
- Run times are not compared. Runs landed on different node types, and builds use `-march=native`.
