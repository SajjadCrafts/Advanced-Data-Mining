[README (9).md](https://github.com/user-attachments/files/30366365/README.9.md)
# MSCS 634 — Project Deliverable 1
## Data Collection, Cleaning, and Exploration

**Dataset:** Clickstream Data for Online Shopping (UCI ML Repository, ID 553)
**Notebook:** `Deliverable_1_Clickstream.ipynb`

---

## Dataset Summary

Browsing activity from an online store selling clothing for pregnant women, covering
April–August 2008. Each record is one click, capturing the product viewed, its price
and colour, where its photo appeared on screen, the page depth reached, and the
country of the originating IP.

| Property | Value |
|---|---|
| Click records | 165,474 |
| Sessions | 24,026 (6.89 clicks each) |
| Attributes | 14 |
| Distinct products | 217 |
| Price range | $18 – $82 (mean $43.80) |
| Source | UCI ML Repository ID 553, CC BY 4.0 |

**Why this dataset:** it supports all four analyses the project requires. `price`
gives a regression target, `main_category` a four-class classification target,
session aggregates the features for clustering — and uniquely, each session is
already a basket of clicks, so association rules can be mined without the artificial
discretisation most datasets require.

---

## Data Cleaning

**1. Semicolon-delimited file.** European convention; the pandas default produces
one malformed column.

**2. `year` dropped** — constant at 2008, zero variance.

**3. Six categoricals stored as integers** — the most serious issue. `country`,
`page 1`, `colour`, `location`, `model photography` and `price 2` are labels encoded
as numbers, so pandas loads them as `int64` and every downstream method treats them
as continuous. Left uncorrected, regression in D2 would impose an arbitrary linear
ordering on unordered categories, and K-Means/k-NN in D3 would compute meaningless
distances. `page` was deliberately **kept** numeric — it is genuinely ordinal.

**4. Date reassembled** from the three components, which also validates them.

**5. Missing values — despite documentation to the contrary.** UCI reports none and
`df.isnull()` agrees; both are wrong. The description file defines **country code 12
as `unidentified`** — an unresolvable IP. Stored as a valid integer in the normal
code range, it escapes `isnull()`, a sentinel scan, and a placeholder-text search
alike. **210 records (0.127%)**, findable only by reading the documentation.
*Treatment:* retain the records, flag via `country_known`, set `country` to `NaN` —
dropping them would discard valid observations of category, colour, price and depth
over an attribute most of the analysis doesn't use.

**6. Duplicates.** Identical content rows aren't errors here (a user can click the
same product twice); the meaningful check is that `(session_id, click_order)` forms
a unique key. It does.

**7. Country imbalance.** Grouped by observed frequency rather than a hard-coded
list.

| # | Issue | Action |
|---|---|---|
| 1 | Semicolon-delimited | `sep=";"` |
| 2 | `year` constant | Dropped |
| 3 | Six categoricals as integers | Recoded; `page` kept ordinal |
| 4 | Date fragmented | Assembled and validated |
| 5 | Country code 12 = unidentified | 210 flagged, `country` → `NaN` |
| 6 | Duplicates | Key verified; content repeats retained |
| 7 | Country imbalance | Grouped by frequency |

---

## Unit of Analysis

The data is at click level but the questions are about sessions. Modelling at click
level would treat rows as independent when clicks within a session are correlated
and come from one visitor — over-weighting long sessions and fitting the most active
visitors rather than the population.

Both representations were built: **click level** for association rules and
page-position analysis, **session level** (24,026 rows) for clustering and
session-outcome modelling. Session features: click count, deepest page, distinct
categories / products / colours, mean / max / min price, price range.

An engineered target `deep_session` (reached page 3+) splits 38% deep / 62% shallow.
Note this is an **engagement proxy, not a purchase outcome** — this dataset records
browsing only.

---

## Outlier Analysis

Assessed at session level using both the IQR rule and z-scores. `n_clicks` is
strongly right-skewed (skewness **4.88**; median 4 clicks, max 195), so the IQR rule
flags a substantial share of sessions.

**Decision: retain all sessions.** A long session is not a measurement error — it's
a visitor who browsed extensively, precisely the behaviour of interest. Removing
them would delete the most informative records and bias results toward casual
browsers.

The skew is a property of the process, not a defect: web session lengths are
characteristically heavy-tailed, and the IQR rule assumes rough symmetry, so here it
flags the distribution's natural shape. Extreme sessions were **flagged** instead,
and a log transform prepared for the scale-sensitive clustering in D3.

---

## Key Insights

**1. Six categoricals stored as integers** was the most serious data quality issue —
it would have corrupted both regression and distance-based methods silently.

**2. The dataset had missing values despite documentation to the contrary.** The
data dictionary is part of the data.

**3. Unit of analysis was the key preparation decision** — 165,474 clicks collapse
to 24,026 sessions.

**4. Traffic is 81% domestic (Poland).** International findings rest on a small
minority of records.

**5. `price` is *completely* determined by `clothing_model`** — all 217 products
have exactly one price, zero within-product variance. Not a correlation, an
identity.

**6. Browsing category strongly predicts session depth**, from 15.3% deep for skirts
to 59.9% for sale — a 44.6-point spread, the widest of any feature examined.

**7. Screen position affects click volume** — top-left leads at 34,532 clicks,
bottom-right trails at 20,743. But the pattern isn't a clean F-shape: bottom-middle
(27,783) beats top-right (21,656), suggesting columns matter more than rows.

**8. There is no purchase outcome in this data**, so no model built on it can
predict conversion.

---

## Challenges Encountered

**Data quality issues that produce no error message.** The integer-coded
categoricals generate no warning, no exception and no nulls — every operation
succeeds and returns a plausible number. Resolved by validating each column against
its *semantic* content rather than its loaded dtype. A dtype that doesn't match what
a column represents is itself a finding, and silent errors are more dangerous than
loud ones.

**Missing values that the documentation denied existed.** Both UCI and `isnull()`
reported a clean dataset. Resolved by reading the data description file, where
country code 12 is defined as unidentified — a check no programmatic profiling would
have performed.

**Detecting price leakage before modelling.** `price` is the obvious D2 target, but
products have fixed prices, so `clothing_model` might simply *be* price. Measuring
within-product variance during EDA confirmed it: all 217 products have exactly one
price. Both `clothing_model` and `price_above_avg` will be excluded from D2.
Catching this during exploration avoided building a deliverable on an R² near 1.0
that would have meant nothing.

---

## Implications for Later Deliverables

**D2 — Regression.** Target `price`; **exclude `clothing_model` and
`price_above_avg`**; predict from category, colour, photo style, position, page
depth. Expect a modest R² — an honest moderate result beats an inflated one from
leakage.

**D3 — Classification / Clustering / Rules.** Target `deep_session` or
`dominant_category`, stratifying splits. Cluster on session features using the log
transform and standardisation (K-Means is scale-sensitive). Mine rules directly on
session baskets — no discretisation required.

**D4 — Ethics.** `country` comes from IP geolocation without explicit consent. More
significantly, the store sells maternity clothing, so browsing behaviour permits
inference of **pregnancy** — a sensitive health status — from data collected for
commercial purposes. The data predates GDPR, and generalises to one Polish retailer
in 2008 rather than to online shopping broadly.

---

## Repository Contents

```
├── Deliverable_1_Clickstream.ipynb           # Analysis notebook
├── README.md
├── e-shop clothing 2008.csv                  # Source data
├── clickstream_clean_click_level.csv         # Cleaned output
└── clickstream_clean_session_level.csv       # Cleaned output
```

## How to Run

```bash
pip install pandas numpy matplotlib seaborn jupyter
jupyter notebook Deliverable_1_Clickstream.ipynb
```

Place `e-shop clothing 2008.csv` beside the notebook, or update `DATA_PATH`.

## References

Łapczyński, M., & Białowąs, S. (2013). Discovering Patterns of Users' Behaviour in
an E-shop. *Studia Ekonomiczne*, 151, 144–153.

*Clickstream Data for Online Shopping* [Dataset]. (2019). UCI Machine Learning
Repository. https://doi.org/10.24432/C5QK7X (CC BY 4.0)
