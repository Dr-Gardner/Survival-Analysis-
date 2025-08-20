# Customer Churn Survival Modeling (2010–2016)

Predict customer churn using survival analysis with **(1) Cox Proportional Hazards** and **(2) Random Survival Forests**.
The project uses **time-based splits** (Train: 2010–2013, Val: 2014–2015, Test: 2016) with **strict leakage control**.

---

## TL;DR

* **Goal:** Early churn risk estimation with interpretable (CoxPH) and non-parametric (RSF) survival models.
* **Data shape:** Transactional rows (`id`, `date`, `amount`, …) + membership dates (`join_date`, `resign_date`), event flag (`event = 1` churn, `0` censored).
* **Models:** CoxPH (with PH checks and optional time-varying effects) and Random Survival Forest (RSF).
* **Metrics:** C-index, time-dependent AUC, Brier/IBS, calibration, KM curves by risk decile.

---

## Data

Each row = one transaction:

| column        | type | description                    |
| ------------- | ---- | ------------------------------ |
| `id`          | int  | customer identifier            |
| `date`        | date | transaction date               |
| `amount`      | num  | spend amount                   |
| `join_date`   | date | membership start               |
| `resign_date` | date | date of churn (NA if censored) |
| `event`       | int  | 1 = churned, 0 = censored      |

---

## Time Splits

* **Train:** 2010-01-01 → 2013-12-31
* **Validation:** 2014-01-01 → 2015-12-31
* **Test:** 2016-01-01 → 2016-12-31

---

## Leakage Control

* Remove IDs who churn in **Train** from **Validation**; remove churners in **Train/Validation** from **Test**.
* Censor non-churners at the split end (e.g., Train censor = **2013-12-31**).

---

## Feature Set (example)

* `base`: average spend per transaction in the split window
* `grats`: gratuity / extra-spend signal
* `txn_count`: number of transactions in the window
* `inter_time`: inter-purchase time summary (e.g., mean days between transactions)
* *(Optional)* recency/frequency/rolling 90-day aggregates computed **up to interval start** for start–stop data
