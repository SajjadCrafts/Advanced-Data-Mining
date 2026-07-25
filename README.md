# MSCS 634 — Project Deliverable 1
## Data Collection, Cleaning, and Exploration

**Dataset:** Clickstream Data for Online Shopping (UCI ML Repository, ID 553)  
**Group Member:** Mohammed Sajjad Mohammed Arif Ansari, Hoai Nam Tran  
**Course:** Advanced Big Data and Data Mining

---

## Dataset

Browsing activity from an online store selling maternity clothing, April–August 2008.
Each row is one click: the product viewed, its price and colour, where its photo sat
on the screen, how deep the visitor went, and their country.

**165,474 clicks · 14 attributes · 24,026 sessions · 217 products ($18–$82)**

We chose it because it covers all four parts of the project: `price` for regression,
`main_category` for classification, session summaries for clustering, and — unusually
— each session is already a basket of clicks, so association rules need no invented
transactions.

## Cleaning

- **Semicolon-delimited file** — needs `sep=";"` to load
- **`year` dropped** — every row says 2008
- **Six categories stored as numbers** — recoded to labels; kept `page` numeric since
  page 3 really is deeper than page 2
- **Date rebuilt** from separate year/month/day columns
- **210 missing values found** — flagged, not deleted
- **Duplicates checked** — none
- **Country grouped** — one market dominates

**On the missing values:** Country code 12 means the IP couldn't be traced, but because it's stored as a
normal number no automatic check finds it. I only found it by reading the
documentation file. I kept those rows and added a `country_known` flag, since the
rest of their data is fine.

## Working at the right level

Each row is one click, but my questions are about sessions. One visitor can produce
50 clicks, so treating clicks as separate would let heavy browsers dominate. I built
both versions — click level for association rules, session level for clustering — and
added a `deep_session` label (reached page 3+, splitting 38/62). It measures
engagement, not purchases; this data has no sales.

## Outliers

The typical session is 4 clicks; the longest is 195. **I kept them all.** A long
session isn't an error, it's someone browsing a lot — exactly what I want to study.
I flagged the extremes and prepared a log-transformed version for clustering.

## Findings

- **Poland is 81% of traffic**, so other-country findings rest on a small slice
- **`price` is fixed per product** — all 217 have one price, so the product code
  doesn't predict price, it looks it up
- **Category predicts browsing depth** — 15.3% deep for skirts vs 59.9% for sale
- **Screen position matters** — top-left 34,532 clicks, bottom-right 20,743

## Problems and Solutions

**Bad data that gives no error.** Categories stored as numbers crash nothing, everything runs and returns reasonable-looking results. I had to check what each
column *means*, not how Python loaded it.

**Missing values the documentation denied.** Only reading the description file found
them.

**Nearly picking a broken target.** `price` looked obvious until we checked whether
products have fixed prices. They do. Leaving the product code in would have given a
near-perfect score that meant nothing.
