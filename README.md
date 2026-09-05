# FlyRank Capstone — CTR / Engagement Opportunity Scoring

## What's in this repo

```
work/
  capstone_ctr_opportunity_scoring.ipynb   <- the pipeline: data -> baseline -> model -> ranked opportunities
  (drop your weekly assignment notebooks here too)
docs/
  index.html                               <- the deployed paper (GitHub Pages serves this)
submission/
  paper_url.txt                            <- must contain exactly one line: your live paper URL
paper_draft.md                             <- markdown source of the paper, mirrors docs/index.html
README.md                                  <- this file
```

## 1. Get data access

Request read access to the FlyRank ML Internship dataset on Hugging Face (gated, ~2 min approval — see the dataset card for the exact repo id). Once approved, generate a **read** token and set it locally:

```bash
export HF_TOKEN=hf_xxxxxxxxxxxx
```

Never commit this token or put it in any notebook cell that gets pushed.

## 2. Confirm the real schema

Open **Starter Notebook 03** first and check the actual table/column names in the warehouse release you were granted. Then open `work/capstone_ctr_opportunity_scoring.ipynb` and edit the `CONFIG` dict at the top (`hf_dataset_glob`, `page_col`, `query_col`, `impressions_col`, `clicks_col`, `position_col`, `date_col`) to match reality — the notebook ships with placeholder names.

## 3. Run it

```bash
pip install duckdb scikit-learn pandas matplotlib nbformat
jupyter notebook work/capstone_ctr_opportunity_scoring.ipynb
```

Run top to bottom. It writes three files into your working directory:
- `ranked_opportunities_holdout.csv` — the ranked action list with reason codes
- `model_vs_baseline_metrics.csv` — MAE/R² for baseline vs. model
- `model_vs_baseline.png` — the comparison chart

## 4. Fill in the paper

Both `paper_draft.md` and `docs/index.html` have clearly marked **TO FILL IN** spots (dashed boxes in the HTML, `*[...]*` in the markdown). Copy your real numbers, the chart, and the top rows of the ranked CSV into both. Before publishing:
- Replace any real page URLs/domains with hashed or generic page IDs (public-safe rule — no client names, domains, or URLs).
- Sanity-check the abstract and results sections say only what your holdout numbers actually support.

## 5. Deploy on GitHub Pages

```bash
git add .
git commit -m "Capstone: CTR opportunity scoring"
git push
```

Then in the repo on GitHub: **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch: main, folder: /docs → Save.**

GitHub will give you a URL like `https://your-username.github.io/your-repo-name/`. Put that exact URL as the single line in `submission/paper_url.txt` and push again.

## 6. Submit

Submit your repo URL wherever the capstone asks for it — the grader finds your paper through `submission/paper_url.txt`.
