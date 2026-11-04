# Preprocessing notes – chiller operating log

**Source.** `Analyse_Chillers_AI_Course.xls`, sheet *Log_Chiller_South*: 522 hourly log records, 5 chillers (2 compressors each), 11–20 Mar 2026, 60 columns. Two title rows precede the header; the row order is descending in time.

## 1. Structure and timestamps
* `Date` + `Time` combined into one timestamp; records sorted ascending. No duplicated (chiller, timestamp) pairs.
* Records are hourly *within* a chiller, but chillers are rotated on and off duty, so each chiller has gaps (longest gap 113 h, Chiller 1). Gaps are an operating pattern, not missing sensor data, and the gaps are never interpolated across.
* The log does not form one regular time series per chiller, so no sequence model was used; LOF operates on single records.

## 2. Units and derived columns
* Temperatures °C, compressor pressures kg/cm², pump pressures bar (mixed units, not converted because pump pressures are not model inputs), currents A, voltage V.
* Many logged columns are differences of others (they hold on most rows) (lift T = discharge T – suction T, lift P, condenser approach = discharge sat. T – condenser outlet, evaporator approach, both superheats) and **running load % = motor current / full-load amps** (2.25 A per % for compressor 1, 2.15 for compressor 2). They add no information and were not used as model inputs.
* Run-hour counters are cumulative meters, with several decreases/jumps (e.g. 1179 or 40715 against ~11 500), so they were audited and excluded.

## 3. Cleaning decisions (rule based; nothing fitted)
| Decision | Rule | Effect |
|---|---|---|
| Implausible value → NaN | engineering range per variable (table in `model_config.json`) | 28 cells in 21 rows, e.g. 2196 A, 631 °C, 40.8 V, 85 °C |
| Compressor off | running load = 0 % | 5 rows kept as a separate operating state and **not scored** (their own variables are legitimately 0) |
| Tower fan 2 off | all three phase currents = 0 | 2 rows, valid state, not treated as errors |
| Load/current mismatch | load differs from current/(A per %) by > 8 points | load rebuilt from current in 6 cells |
| Condenser pump pressures | 0.0 on ~5 % of rows with normal pump current | excluded from features |

## 4. Imputation (training statistics only)
1. Causal forward fill from the same chiller's previous record when ≤ 3 h old.
2. Otherwise the training-block median of that chiller, then the global training median.
Only 11 cells (0.09 % of scored cells, 8 rows) needed imputation. None of the top-15 cases contains an imputed value.

## 5. Features, pruning and scaling
* 24 candidate features from independent measurements/differences in five subsystem groups; pruning at training |Spearman| > 0.90 dropped exv_dp2 → **23 model features**.
* `RobustScaler` (median/IQR) fitted on the training block.
* No dimensionality reduction (PCA would make explanations unreadable).

## 6. Chronological split and leakage control
Train 11–15 Mar (282 scored rows), validation 16–17 Mar (115), test 18–20 Mar (120). Imputation medians, pruning, scaling, the A-per-% constant, the LOF fit and the alert threshold use training rows only; the validation block was used only to choose k; the test block was scored once. For the synthetic-injection check the anomalies were injected into a separate copy of the test inliers while the clean reference data stayed unchanged.

## 7. Known limitations
No kW or flow data (efficiency is judged through lift/ideal-COP proxies); no fault labels; only 10 days of data; chillers are pooled; one-hour duty events and shared sensors create near-duplicate records.
