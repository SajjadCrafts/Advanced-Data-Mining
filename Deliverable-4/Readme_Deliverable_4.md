# Discovering Patterns in Online Shopping Behavior

**A Data Mining Analysis of E-Shop Clickstream Data**

**Team:** Hoai Nam Tran, Mohammed Sajjad Mohammed Arif Ansari  
**Course:** MSCS-634-M20 — Advanced Big Data and Data Mining  
**Instructor:** Dr. Satish Penmatsa, University of the Cumberlands

---

## Dataset

Clickstream Data for Online Shopping (UCI ML Repository, ID 553): browsing activity from a
maternity-clothing retailer over five months in 2008 — 165,474 clicks across 14 attributes,
aggregated to 24,026 sessions. The data records browsing only, no purchases.

## Project Steps

- **Deliverable 1 — Cleaning & Exploration:** Fixed six mis-typed categorical columns, found
  hidden missing values (country code 12 = unresolved IP), dropped a zero-variance column, and
  aggregated clicks into sessions. Discovered every product has one fixed price, which shaped
  the next step.
- **Deliverable 2 — Regression:** Since price can't be predicted from the product itself, we
  instead predicted product **attention** (clicks received). A depth-limited Random Forest
  performed best (R² = 0.46). Higher-priced products got more attention, not less.
- **Deliverable 3 — Classification, Clustering, Pattern Mining:** Predicted session engagement
  from the first two clicks (best model: tuned Decision Tree, 64% accuracy). Clustered sessions
  into four shopper types (budget bouncers, premium bouncers, moderate browsers, deep
  explorers). Found that shoppers browse across categories and mix price tiers within a
  session (association rule mining).
- **Deliverable 4 — Synthesis:** Combined all findings into a final report and presentation,
  with practical recommendations and an ethics discussion.

## Key Findings

- Page position is the strongest driver of attention.
- Higher-priced products attract more attention; discounting isn't the main driver of
  engagement.
- The first two clicks of a session carry real (if modest) signal about eventual engagement.
- Shoppers naturally fall into four behavioral segments — notably, two very different types
  of "bouncer" that a simple bounce-rate metric would treat as the same.
- Shoppers browse across product categories and price tiers in the same session.

## Recommendations

1. Lead merchandising with premium items, not discounts.
2. Recommend across categories ("complete the outfit"), not just similar items.
3. Treat budget and premium bouncers differently.
4. Target deep-explorer sessions for loyalty features.
5. Tune early-engagement alerts toward recall, not just accuracy.




