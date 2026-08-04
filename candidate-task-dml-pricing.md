# Causal ML — price elasticity for pricing optimisation (mid-level, harder tier)

**Time:** 50–60 minutes.
**Tools:** open book, including AI assistants — use whatever you'd normally use.
**Stack:** Python, pandas, numpy, scikit-learn, statsmodels (or your own OLS/vectorized code).

---

## Context

The pricing team wants to know: **if we raise a product's price by 1%, what happens to units sold?** That number (the price elasticity) feeds directly into pricing optimisation — it tells you whether raising price on a given product grows or shrinks revenue.

You're given historical SKU-level data. Prices in this data were **not** set randomly — the pricing team already reacts to signals like seasonality, stock levels, and competitor pricing when they set price. That means price and demand share common causes, and naively regressing demand on price will not give you the causal effect of price.

Your job: estimate the causal price elasticity properly, using **Double Machine Learning (DML)**.

---

## Data

**`pricing_data.csv`**

| Column               | Description                                             |
| --------------------- | --------------------------------------------------------- |
| `sku_id`              | Product identifier                                       |
| `category`            | Product category                                         |
| `price`               | Price at time of sale                                    |
| `units_sold`          | Units sold                                                |
| `seasonality_index`   | A seasonal demand signal, 0–1                             |
| `stock_level`         | Units on hand                                             |
| `competitor_price`    | Nearest competitor's price for a comparable product       |
| `promo_flag`          | 1 if the SKU was on promotion                             |

---

## Requirements

### Part A — Naive baseline (~10 min)

Regress `log(units_sold)` on `log(price)` with no other controls. Report the coefficient. This is your "what a dashboard would show you if you weren't careful" baseline — you'll come back to it.

### Part B — Double ML (~30–35 min)

Implement the DML partialling-out approach:

1. Using `seasonality_index`, `stock_level`, `competitor_price`, `promo_flag`, and `category` as controls, fit a flexible model (e.g. gradient boosting / random forest) predicting `log(price)` from the controls.
2. Fit a second flexible model predicting `log(units_sold)` from the **same controls** (not including price).
3. Get **out-of-sample residuals** for both — this must be done with proper K-fold cross-fitting: each row's residual has to come from a model that was **not** trained on that row.
4. Regress the demand residuals on the price residuals (OLS). The coefficient is your causal elasticity estimate, θ.
5. Report θ with a confidence interval.

### Part C — Compare and interpret (~10–15 min)

- How does θ compare to your Part A naive estimate? What does that difference tell you about how the pricing team has historically set prices?
- What would go wrong if, in step 3, you fit your models and computed residuals on the **same** rows (no cross-fitting)? You don't have to prove it mathematically — describe what you'd expect to observe and how you'd catch it if it happened.
- Is your θ a sensible causal elasticity for a normal good? What would make you distrust it?
- How would you use this number for actual pricing decisions — what's still missing between "we estimated an elasticity" and "we changed a price"?

---

## Out of scope

- Heterogeneous treatment effects / per-SKU elasticity (i.e. `CausalForestDML`-style estimators)
- Instrumental variables
- Formal asymptotic standard errors for the cross-fitted estimator (bootstrap or the naive OLS SE on stacked residuals is fine)

Good luck.
