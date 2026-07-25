# MSCS 634 — Project Deliverable 2
## Regression Modeling and Performance Evaluation

**Dataset:** Clickstream Data for Online Shopping (UCI ML Repository, ID 553)  
**Group Member:** Mohammed Sajjad Mohammed Arif Ansari, Hoai Nam Tran  
**Course:** Advanced Big Data and Data Mining

---

## The question

Deliverable 1 planned to predict `price`, but that isn't a useful question, the
store already knows what it charges. The useful question runs the other way:

 **Which products attract customer attention, and does price play a part?**

**Target:** how many clicks a product received.
**Features:** price, category, colour, screen position, photo style, page.

## Why product level

Each row in the raw data is a click, not a product. Since the question is about
products, We need one row per product.

There's also no choice about it. Every product attribute: colour, position, photo
style, page, category — never changes for a given product. So all 165,474 clicks
contain only **217 distinct combinations**. Modelling at click level would feed the
same 217 rows to the model thousands of times, faking a large sample without adding
real information.

Grouping by product fixes this, and the click counts become the target.

## What "attention" means?

`n_clicks` measures interest at the browsing stage: someone chose to look closer.
It does **not** measure purchases, since this dataset has no sales data at all.

Products average 763 clicks from 684 distinct visitors, about 1.1 clicks per person.
So attention here is mostly **reach** (many different people looked) rather than
repeat viewing.

## Feature engineering

Four features added to the raw attributes:

- **`price_band`** — price quartile, so the model can capture non-linear price
  effects
- **`is_sale_item`** — sale items may behave differently
- **`is_top_row`** — tests vertical position separately from the six-way position
- **`clicks_per_session`** — repeat viewing (kept for interpretation only, *not* used
  as a predictor, since it comes from the target)

That gives about 28 features for 217 rows — roughly 8 rows per feature, small enough
that regularization does real work.

## Models and results

Four models, identical preprocessing, 5-fold cross-validation. Regularization
strength and tree depth were tuned rather than guessed.

| Model | R² | RMSE | R² std |
|---|---|---|---|
| **Random Forest (depth=2)** | **0.457** | 426 | 0.085 |
| Lasso (α=20) | 0.320 | 479 | 0.098 |
| Ridge (α=20) | 0.316 | 477 | 0.133 |
| Linear Regression | 0.177 | 516 | 0.249 |

**Plain Linear Regression is worst and least stable.** Its per-fold scores were
`[0.34, 0.12, 0.36, 0.36, −0.29]` — one fold went negative, which is textbook
overfitting on 217 rows.

**Ridge and Lasso nearly double it.** With ~7 rows per feature the unconstrained
model was fitting noise, and shrinking the coefficients fixed it. Lasso dropped 27 of
34 features entirely and still matched Ridge.

**Random Forest wins, but only when shallow.** Unrestricted, it falls below Ridge.

## Key findings

**Page position dominates everything else.** `page_number` is the strongest driver in
both models, the largest Ridge coefficient and 0.67 of the Random Forest's total
importance, roughly five times the next feature. Products deeper in the site get far
fewer clicks.

This measures *exposure* rather than appeal — a page-5 product can't attract
attention because few visitors reach it. But it's also the most actionable finding
here, since page placement is something the retailer controls directly.

**Higher prices attract more attention, not less** — correlation +0.26, the opposite
of intuition. The effect is not monotonic:

| Price band | Mean clicks |
|---|---|
| Budget | 573 |
| Mid-low | 892 |
| Mid-high | 700 |
| **Premium** | **1,053** |

Attention rises, dips, then peaks. A straight line can't bend twice, which is why the
linear models plateau near 0.32 while the tree reaches 0.46.

**Category comes second.** Trousers and skirts draw about twice the attention of sale
items. Sale is the least-viewed category despite being cheapest, which, with the
premium finding, argues against discounting as an engagement lever.

**Colour effects are unreliable.** Ridge gives blue a large positive coefficient, but
the Random Forest barely uses colour at all. With 14 colours across 217 products,
each covers few items, so these are likely noise. The disagreement between the two
models is itself the warning sign.

## Limitations

- **217 observations is small.** CV standard deviations run ±0.09 to ±0.25, so model
  differences are real but not precisely measured.
- **Attention is not sales.** No purchase data exists here, so none of this is a
  revenue recommendation.
- **Correlation, not causation.** Premium products may draw attention because they're
  premium, or because they're also newer or better placed. The data can't separate
  these.

## Challenges and solutions

**The original target was unusable.** `price` passed Deliverable 1's leakage check
but failed a deeper one: not just the product code, but *every* remaining feature is
fixed per product.

**Reframing rather than patching.** Instead of fixing the validation, We changed the
question. Predicting attention makes the product the natural unit, so leakage can't
happen.

**A small sample after aggregation.** 165,474 rows became 217. Resolved by tuning
regularization properly and reporting fold-level variance rather than just averages.

**Two models disagreeing about colour.** Rather than picking whichever supported a
nicer story, We treated the disagreement as evidence the colour effects aren't
trustworthy.


