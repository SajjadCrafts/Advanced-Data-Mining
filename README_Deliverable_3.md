# MSCS 634 — Project Deliverable 3
## Classification, Clustering, and Pattern Mining

**Dataset:** Clickstream Data for Online Shopping (UCI ML Repository, ID 553)
**Group Members:** Mohammed Sajjad Mohammed Arif Ansari, Hoai Nam Tran
**Course:** Advanced Big Data and Data Mining

---

## Overview

Deliverable 2 asked which *products* attract attention. This one moves to *visitors*:

- **Part A — Classification.** Can we tell from a visitor's first two clicks whether
  they'll browse deeply? *(Decision Tree, k-NN, Naive Bayes)*
- **Part B — Clustering.** What types of shopping session exist? *(K-Means)*
- **Part C — Pattern mining.** Which products get browsed together? *(Apriori)*

## Part A — Classification

- **Target:** did the session reach page 3 or deeper?
- **Features:** the **first two clicks only** — what they landed on, its price,
  colour, photo position, plus country and weekday
- **Why only two clicks:** predicting depth from a whole session is circular, since a
  long session is deep by definition. Two clicks makes it a real prediction, and one
  a retailer could act on while the visitor is still browsing
- 18,984 sessions qualified; 45.6% were deep, so the baseline is 54.4% accuracy

**Tuning.** The Decision Tree was tuned with `GridSearchCV` over `max_depth`,
`min_samples_leaf`, and `criterion`, scored on **F1 rather than accuracy** so the
metric wouldn't reward favouring the majority class. Best: `entropy`, depth 10,
min leaf 10 — lifting F1 from 0.545 to 0.569.

**Results:**

- **Decision Tree (tuned)** — accuracy 0.636, F1 0.567, AUC 0.669
- **k-NN (k=25)** — accuracy 0.625, F1 0.542, AUC 0.663
- **Naive Bayes** — accuracy 0.596, F1 0.498, AUC 0.628

All beat the baseline, so two clicks carry real but modest signal. The tree catches
73% of shallow sessions and only 52% of deep ones, worth noting, since missing an
engaged visitor costs more than prompting one unnecessarily.

## Part B — Clustering

K-Means on session behaviour, using standardised features and `log_n_clicks` (raw
counts are heavily skewed, so a few 195-click sessions would pull the centres).

**Choosing k.** Silhouette preferred k=2 (0.360 vs 0.272), but that only splits
"short" from "long", something `n_clicks` already tells us. We chose **k=4**, which
separates on engagement *and* price. Silhouette measures how cleanly separated
clusters are, not how useful they are.

**The four session types:**

- **Budget bouncers** (~5,750) — ~2 clicks, cheapest items (~$36), one category
- **Premium bouncers** (~4,600) — ~2 clicks, priciest items (~$55), one category
- **Moderate browsers** (~8,400) — ~6 clicks, 2 categories, mid prices
- **Deep explorers** (~5,300) — ~18 clicks, 3 categories, 7 colours, wide price range

The two brief-visit groups **aren't the same segment**. Both leave after ~2 clicks, so
a bounce-rate report would merge them, but one is price-shopping and the other isn't.
That's exactly what k=2 would have hidden.

## Part C — Association Rule Mining

Each session is a basket of everything browsed. Mining was restricted to **category
and price band** — attributes that vary within a session across products, so
co-occurrence reflects visitor choice rather than the fixed catalogue structure.

**Top patterns:**

- blouses + skirts → mid-high price (lift 1.79, confidence 0.89)
- blouses → budget + mid-high (lift 1.76)
- blouses + sale → mid-high price (lift 1.71)
- trousers + mid-high → blouses (lift 1.69)

**1. Shoppers browse across categories** at lift 1.65–1.79. They aren't looking for
"a skirt" — they're assembling outfits.

**2. Shoppers mix price tiers**, with budget and mid-high items co-occurring at lift
1.76.

Finding 2 looks like it contradicts the clustering, which separated budget from
premium bouncers. It doesn't, bouncer sessions are too short to mix anything, while
these rules come from longer sessions. Both are true of different visitors.

## Real-world applications

- **Classification** — real-time intervention: offer help to visitors predicted to
  disengage. The threshold matters more than the accuracy.
- **Clustering** — budget and premium bouncers need different treatment: price
  reassurance for one, quality or delivery info for the other. Deep explorers are the
  natural audience for recommendations.
- **Pattern mining** — argues for cross-category recommendations ("complete the
  outfit") over more-of-the-same suggestions.

Across all three, the interesting variation is between *visitors*, not within the
catalogue.

## Limitations

- **No purchase data** — a "deep" session is engaged, not necessarily valuable
- **Modest performance** — AUC 0.67 is real but not strong; two clicks is little
  information
- **Cluster count is a judgement call** — the metric preferred k=2, we chose k=4

## Challenges

- **Avoiding a circular target.** Using full-session features would have scored well
  and meant nothing. Restricting to the first two clicks cost accuracy but made the
  result real.
- **A metric that disagreed with usefulness.** Silhouette favoured k=2, which only
  recovered what `n_clicks` already showed. We chose k=4 and documented why.
- **Two results appearing to contradict.** Clustering separated price segments while
  the rules showed mixing. Rather than dropping one, we found they describe different
  session lengths.

