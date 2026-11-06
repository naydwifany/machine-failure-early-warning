# Assumptions and Questions for the Data Owner

The brief asks that any ambiguity about the timing or lineage of a field be documented,
together with the assumption taken. This document covers both.

---

## Assumptions taken

| # | Item | Assumption | Impact if wrong |
|---|---|---|---|
| 1 | `vibration_mm_s` valued `-999` (10 rows) | A "no data" code, not a measurement. Converted to missing. | Low. The value is physically impossible, so this is almost certainly correct. |
| 2 | Missing values in `vibration_mm_s` (103) and `bearing_temp_c` (104) | Missing at random. Left as missing and handled natively by the model. | Medium. If values are missing *because* of machine condition, that pattern is itself informative and should become a feature. |
| 3 | 80 rows sharing `machine_id` + `observation_timestamp` | Left in place, not deduplicated. | Medium. If these are a join error, duplicated rows could land on both sides of the split and inflate the score. |
| 4 | `machine_class` valued `unknown` (15 rows) | Missing data disguised as a category. Treated as its own category. | Low. Only 0.1% of the data. |
| 5 | Low base rate in January 2024 (0.8% vs 4–7% in other months) | The 14-day lookback columns are not yet fully populated at the start of the observation period. | Low. That month falls on the training side, not the test side. |
| 6 | Train/test boundary at the 70th percentile of observation time | Gives an 18-week test window, long enough to judge stability. | Low. Checks with other boundaries lead to the same conclusion. |
| 7 | One observation equals one inspection opportunity | Used to compute "alerts per week". | Medium. If several observations of one machine in a week should count as a single visit, the real alert load is lower. |

---

## Questions for the data owner

Ordered by how much the answer affects the model's viability.

### 1. When is `corrective_orders_14d` populated?

Is this column filled at the moment the observation is recorded, or backfilled once a
failure is known?

**Why this comes first.** The answer determines whether the 0.697 score is valid. The
analysis shows this column jumping from exactly 0.000 to exactly 1.000 at the same
observation as the inspection-interval change — a pattern consistent with it being written
alongside a human decision rather than observed independently.

### 2. Who changes `inspection_interval_days`, and on what basis?

Is the interval change a human decision triggered by suspicion about machine condition? Is
the change recorded with its own timestamp, separate from the observation?

### 3. Does `unplanned_stops_14d` really count back 14 days from the observation?

If the window is anchored to some other date — month end, or a reporting date — this column
also carries information from the future.

### 4. What does `unknown` mean in `machine_class`?

A genuine fourth machine class, or missing data?

### 5. Are the 80 identical pairs legitimate?

Two real readings at the same second, or a join error?

---

## One number that needs to be agreed

Before the next model is built, the process owner needs to supply a **cost ratio**: how
many times more expensive is one missed failure than one wasted inspection.

Without that number, the alert threshold cannot be set responsibly. An analyst can present
the full trade-off between recall and precision, but cannot choose the point on it — that
choice belongs to the process owner.
