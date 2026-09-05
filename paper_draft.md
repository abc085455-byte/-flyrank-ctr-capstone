# Finding Under-Clicked Winners: A CTR Opportunity Score for Ranked Content

## Title + Abstract
Pages that rank well don't always get clicked at the rate their position predicts — this leaves organic clicks on the table without any ranking change needed. We build a position-adjusted expected-CTR model (gradient-boosted regression vs. an isotonic position-only baseline) on 199,693 published pages from the FlyRank search warehouse, validated on a page-level holdout split to avoid leakage. The model improved variance explained over the baseline (holdout R² 0.057 vs. 0.034) but did not improve average error (MAE 0.00309 vs. 0.00304) — a genuinely mixed result, reported honestly rather than oversold. We use the model's expected-CTR estimates to rank all qualifying pages by estimated lost clicks and assign each a reason code and recommended action (metadata rewrite, content review, or monitor). The result is a prioritized action list a content/SEO team can work top-down rather than guessing which of thousands of pages to touch first.

## Introduction / Problem Statement
Search teams often audit content by raw CTR, which conflates "bad snippet" with "bad position" — a #9 result and a #1 result have structurally different CTR ceilings, so raw CTR alone misdirects effort. This work supports a concrete decision: **which pages should a content/SEO team review first, and what kind of review (metadata vs. content vs. none)?**

## Data
- Source: FlyRank ML Internship dataset, `FlyRank/internship-warehouse` on Hugging Face (Hub gated release, read-token access).
- Tables used: `dim_content` (content metadata, filtered to `is_published = TRUE` and `is_deleted = FALSE`), `fact_content_daily_performance` (18 monthly partitions, January 2025 through June 2026), and `fact_content_query_90d` (used only to derive query breadth, `n_queries`, per page).
- Date window: full available range across the monthly partitions, January 2025 – June 2026.
- Aggregation level: page (`content_hash_id`), not row/impression level — impressions and clicks summed across the whole window per page, then joined to query-count.
- Exclusions: pages with fewer than 50 total impressions (noise-level) were dropped, leaving 199,693 qualifying pages.
- All identifiers in the source data are pre-hashed (`content_hash_id`, `client_hash_id`, etc.) — no client names, domains, raw URLs, or credentials appear anywhere in this paper or its linked repo.

## Methodology
- **Label**: page-level CTR = clicks / impressions, aggregated over the observed window.
- **Baseline**: isotonic regression of CTR on rounded average position (impression-weighted, clipped to positions 1–20), fit only on the training split — captures the well-known "expected CTR falls off with position" curve without any other features.
- **Model**: gradient-boosted regressor (`max_depth=3`, `n_estimators=200`, `learning_rate=0.05`) using average position + query breadth (`n_queries`) as features.
- **Validation design**: grouped split **by page** (not by row), 70/30 train/holdout (139,785 train pages, 59,908 holdout pages), so no page's own performance leaks into the curve or model used to score it.
- **Leakage checks**: baseline curve and model are both fit exclusively on the training pages; all reported metrics below are holdout-only.

## Results

![Model vs baseline](model_vs_baseline.png)

| Metric | Baseline (position-only) | Model (GBR, position + n_queries) |
|---|---|---|
| MAE (holdout) | 0.00304 | 0.00309 |
| R² (holdout) | 0.034 | 0.057 |

The result is genuinely mixed, and we report it as such rather than picking whichever metric looks better. Adding query breadth as a feature raised the share of variance explained (R² nearly doubled, from 0.034 to 0.057) but did **not** reduce average absolute error — the model's MAE was marginally *higher* than the simple position-only curve. In practical terms: the model is somewhat better at capturing which pages deviate most from expectation, but on a typical page it is not more accurate than just reading off the position curve. Query breadth alone is not a strong enough signal to justify the added model complexity on its own; it is retained here because it feeds the ranking below, not because it clearly beats the baseline.

## Limitations & Honest Framing
- This is a **snapshot** analysis (single aggregation window), not a causal or time-series claim — findings are *observed*, not *directional* trend claims, unless a time-aware version is added.
- CTR opportunity scores are **decision-support**, not proof of a fixable issue — a low score can also reflect factors outside metadata (SERP features, intent mismatch, brand recognition).
- No claim is made about Google's ranking algorithm or about causal refresh impact.
- Sample after the impression-floor filter: 199,693 pages (139,785 train / 59,908 holdout). The model's own validation shows only modest R² (0.057) — most variance in CTR is not explained by position and query breadth alone, so treat individual page-level "expected CTR" values as a rough prior, not a precise target.
- Because the model shows a mixed result versus the baseline (better R², slightly worse MAE), the ranked list below should be read as **directional prioritization**, not a precise prediction of exact recoverable clicks.

## Ranked Recommendations

Top holdout-page opportunities by estimated lost clicks (`(expected_ctr − actual_ctr) × impressions`):

| Page | Avg. position | Impressions | CTR | Expected CTR | Est. lost clicks | Reason | Action |
|---|---|---|---|---|---|---|---|
| content_8c6d8360... | 7.71 | 99,290 | 0.045% | 2.10% | 2,039 | Ranking well, CTR lagging | content_review |
| content_b64fb5a4... | 5.21 | 758,188 | 0.151% | 0.36% | 1,594 | Ranking well, CTR lagging | content_review |
| content_39e19a3e... | 9.72 | 425,137 | 0.003% | 0.36% | 1,508 | Ranking well, CTR lagging | content_review |
| content_e241d641... | 4.10 | 1,829,204 | 0.312% | 0.38% | 1,257 | High visibility, under-clicked | rewrite_metadata |
| content_d0acf706... | 2.22 | 312,069 | 0.049% | 0.39% | 1,062 | High visibility, under-clicked | rewrite_metadata |
| content_93aadc6e... | 4.48 | 498,907 | 0.157% | 0.37% | 1,054 | High visibility, under-clicked | rewrite_metadata |
| content_80eb6221... | 2.81 | 546,198 | 0.303% | 0.50% | 1,054 | High visibility, under-clicked | rewrite_metadata |
| content_300fd9ae... | 5.62 | 362,425 | 0.056% | 0.31% | 909 | Ranking well, CTR lagging | content_review |
| content_dcddbd42... | 5.24 | 109,617 | 0.160% | 0.94% | 858 | Ranking well, CTR lagging | content_review |

*(Full ranked list of all 59,908 holdout pages, and the equivalent run on the full warehouse, is in `work/ranked_opportunities_holdout.csv` in the repo.)*

General playbook derived from the reason codes:
1. **High visibility, under-clicked** → rewrite title/meta description first; cheapest fix, fastest to test.
2. **Ranking well (top 10), CTR lagging** → content/snippet review — check if the page still matches query intent.
3. **On/above expected** → protect as-is; don't touch what's already outperforming its position.
4. **Low visibility and under-clicked** → monitor only; lower priority given limited exposure.

## Reproducibility
- Notebook: `work/capstone_ctr_opportunity_scoring.ipynb` (this repo)
- All weekly assignment notebooks: `work/`
- Data access: gated Hugging Face release, read-token access — see dataset card for request instructions.

## Acknowledgments & Data Credit
Built on the FlyRank ML Internship dataset — [flyrank.ai](https://flyrank.ai).
