# Introduction

This project is an **experimentation audit**, not a predictive model. The
goal isn't to build an uplift/CATE model that predicts which individual
customers are most likely to respond to an email — that's the usual
treatment of this dataset. Instead, this project asks a narrower,
business-first question: given a campaign that already ran, can we trust
the result enough to act on it, and is the result actually worth acting
on?

**The data:** Kevin Hillstrom's MineThatData email challenge — a real,
randomized marketing experiment where ~64,000 customers were split into
three groups (`Mens E-Mail`, `Womens E-Mail`, `No E-Mail`), with each
customer's pre-campaign purchase history and three tracked two-week
outcomes: did they visit, did they convert, and how much did they spend.

**The business question:** a growth team wants to win back dormant
customers with a targeted email. Before recommending this campaign be
scaled up, two things need to be true: the effect has to be real (not
noise, not an artifact of comparing unbalanced groups, not inflated by
running several tests at once), and the effect has to be a genuine
revenue win, not just an engagement bump that doesn't translate into
actual purchases — the "local maxima" trap, where a metric improves while
quietly failing to move the metric that actually matters.

**The decision this analysis supports:** should the company continue and
scale this win-back email program to its full dormant customer base, and
if so, which version — the Mens-styled or Womens-styled creative —
should be prioritized? Answering that well requires more than a single
significance test: it requires checking that the underlying experiment
was run correctly, that there's enough data to trust a null result if one
shows up, that testing multiple outcomes at once hasn't inflated the
chance of a false win, that the revenue estimate is as precise as
possible, and that competing explanations (like customers who would have
come back anyway) have been ruled out before crediting the campaign with
the full effect. That audit — not a prediction model — is what this
notebook walks through, end to end.
## Dataset

Kevin Hillstrom's MineThatData email challenge — a real, randomized
marketing experiment. ~64,000 customers split into three arms
(`Mens E-Mail`, `Womens E-Mail`, `No E-Mail`), with real pre-campaign
customer features and three tracked post-campaign outcomes.

| Field | Meaning |
|---|---|
| `Recency` | Months since customer's last purchase |
| `History_Segment`, `History` | Binned / actual dollar amount of historical spend (pre-campaign) |
| `Mens`, `Womens` | 1/0 — purchased from that department in the past |
| `Zip_Code`, `Newbie`, `Channel` | Other pre-campaign customer features |
| `segment` (treatment) | `Mens E-Mail` / `Womens E-Mail` / `No E-Mail` |
| `visit` | 1/0 — visited site in the 2 weeks following the campaign |
| `conversion` | 1/0 — purchased in the 2 weeks following the campaign |
| `spend` | Dollars spent in the 2 weeks following the campaign |

Loaded via `scikit-uplift`'s `fetch_hillstrom(target_col='all')`, which
pulls the raw CSV from a public, unauthenticated source
([scikit-uplift docs](https://www.uplift-modeling.com/en/latest/api/datasets/fetch_hillstrom.html)).

## Methods 

| Step | Method | Day |
|---|---|---|
| 1 | Distribution-shape check on `spend`/`History` before trusting any mean-based test 
| 2 | Define the dormant cohort from `Recency` 
| 3 | SRM check (chi-square goodness-of-fit) on the 3-way split — overall **and** within the dormant cohort 
| 4 | Power / MDE check on the dormant cohort's actual per-arm sample size 
| 5 | Six simultaneous tests (visit / conversion / spend × 2 treatment arms) with Benjamini-Hochberg correction
| 6 | CUPED on `spend`, using pre-campaign `History` as the covariate
| 7 | Regression-to-the-mean check specifically on the dormant cohort 
| 8 | Guardrail synthesis: does the visit lift survive once spend and the above checks are applied? 


## Repo structure

```
winback-guardrail-ab-test/
├── README.md
├── requirements.txt
├── data/
│   └── raw/                  # created on first run, gitignored
├── notebooks/
│   └── winback_guardrail_analysis.ipynb
└── figures/
```

## Reproducing this

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook notebooks/winback_guardrail_analysis.ipynb
```

Run it top to bottom. The first code cell downloads the dataset (needs an
internet connection once) and caches a local copy in `data/raw/`.

## Status

All eight steps are scaffolded in the one notebook, in order:

- [x] 1 — Pull & inspect
- [x] 2 — Dormant cohort definition
- [x] 3 — SRM check
- [x] 4 — Power/MDE check
- [x] 5 — Six-test Benjamini-Hochberg correction
- [x] 6 — CUPED on spend
- [x] 7 — Regression-to-the-mean check
- [x] 8 — Guardrail synthesis 
