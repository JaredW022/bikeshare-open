# AUDIT 

## GitHub
GitHub: https://github.com/dsouza-dylan/DSC180A-Methodology-6

## Claude's Audit Results
Audit Results

1. Target Leakage (casual, registered, cnt)

src/config.py lines 27-31: LEAKAGE_COLS = {"casual", "registered", "cnt"} documents that casual + registered == cnt exactly.

src/features.py line 88: feature_columns() filters these: [c for c in numeric if c not in FORBIDDEN].

✅ Verified: Forbidden columns are excluded from the feature set by construction (line 86-88).

2. Temporal Ordering of Shift Operations

src/features.py lines 54-80:
```python
out = reindex_to_calendar(...).copy()
# ... features ...
out["cnt_lag_1"] = out[TARGET].shift(1)        # line 78
out["cnt_lag_7"] = out[TARGET].shift(LAG_WARMUP)  # line 79
out["cnt_roll_7"] = out[TARGET].shift(1).rolling(LAG_WARMUP).mean()  # line 80
return out.dropna().reset_index(drop=True)   # line 82
```

⚠️ Finding: Line 78-80 shift BEFORE dropna, but the shift is on the already calendar-reindexed dataframe. Since reindex_to_calendar() (line 43-50) creates a complete calendar with explicit NaN rows for missing days, and we're calling .dropna() at line 82 AFTER building lags, the lag shifts are computed on the full-calendar dataframe where gaps are marked as NaN rows.

However, diff(1) and shift(1) operate on pandas row index positions. Since reindex_to_calendar fills in ALL missing days with NaN (see line 43-50), there are no calendar gaps at shift time—only explicit NaN rows representing missing data. This is correct behavior per the docstring at line 37-42: "pandas .shift(1) moves by rows, not by days... After reindexing, the gap is an explicit NaN that dropna() removes".

✅ Verified: Lags are computed on a complete calendar (post-reindex), shifted before aggregation.

3. Calendar Gaps (temp_change_1)

src/features.py lines 72-73:
out["temp_change_1"] = out.temp.diff(1)

diff(1) is row-based, applied after reindex_to_calendar. Since missing days are inserted as explicit NaN rows and then removed at line 82 via .dropna(), the diff uses only calendar-adjacent rows where both dates exist. The docstring at lines 70-73 confirms: "across the 2011-03-10 gap it yields NaN" because that day itself is a NaN row (never in raw data), so no prev/next exists for it—dropna() removes those rows anyway.

✅ Verified: diff(1) operates on calendar after reindex, inputs to temp_change_1 are known at day's end (temp measured same-day).

4. Silent Row Loss

src/features.py line 82: return out.dropna().reset_index(drop=True)

⚠️ Finding: Line 82 drops NaN rows WITHOUT asserting bounds. The test test_only_expected_rows_are_dropped in tests/test_contract.py (line 28-43) checks that lost rows ≤ LAG_WARMUP + LAG_WARMUP × missing_days, but the code itself has no assertion.

The loss is expected for:
- First LAG_WARMUP days have no complete history
- Missing calendar days cost LAG_WARMUP rows each (from dropna on lags)

✅ Verified by test: tests/test_contract.py line 39-43 bounds this row loss. No assertion in the code itself, but the test enforces it.

5. Baselines Report Completeness

src/train.py lines 48-53:
MODELS = {
    "baseline_mean": DummyRegressor(strategy="mean"),
    "baseline_persistence": PersistenceBaseline("cnt_lag_1"),
    ...
}

✅ Verified: Both mean baseline (line 49) and persistence baseline (line 50, yesterday's count) are implemented.

src/train.py lines 78-84: Models are fitted on train only (model.fit(train[cols], ...)), then predictions made on valid/test separately.

✅ Verified: Train/valid/test split by date (lines 56-62), fitting happens on train rows only, no leakage into validation/test during fit.

6. Baselines in Saved Metrics

reports/metrics.json does not exist (directory even doesn't exist yet).

⚠️ Finding: Cannot verify saved metrics report both baselines. The fit_and_score() function (line 65-86) computes and stores results for all models including "baseline_mean" and "baseline_persistence", but since the reports directory doesn't exist, we cannot confirm what was actually saved to disk.

The code path:
- Line 78: model.fit(train[cols], train[TARGET]) - fits on training data only
- Lines 79-84: Creates results dict with model name → split (valid/test) → metrics

This should produce R² scores for both baselines plus ridge/hgb. The issue is purely that the output file doesn't exist to verify it was written correctly.

---

Summary

┌────────────────────────────────────┬───────────────┬─────────────────────────────────────────────┐
│               Check                │    Status     │                    Notes                    │
├────────────────────────────────────┼───────────────┼─────────────────────────────────────────────┤
│ Forbidden columns excluded         │ ✅ Pass       │ feature_columns() filters FORBIDDEN         │
├────────────────────────────────────┼───────────────┼─────────────────────────────────────────────┤
│ Lags on complete calendar          │ ✅ Pass       │ reindex_to_calendar fills gaps before shift │
├────────────────────────────────────┼───────────────┼─────────────────────────────────────────────┤
│ Rolling excludes forecast day      │ ✅ Pass       │ shift(1) before rolling (line 80)           │
├────────────────────────────────────┼───────────────┼─────────────────────────────────────────────┤
│ Row loss bounded by tests          │ ⚠️ Partial    │ Code lacks assertion, test bounds it        │
├────────────────────────────────────┼───────────────┼─────────────────────────────────────────────┤
│ Chronological split                │ ✅ Pass       │ temporal_split() uses dates not random      │
├────────────────────────────────────┼───────────────┼─────────────────────────────────────────────┤
│ Baselines implemented              │ ✅ Pass       │ Both mean and persistence in MODELS         │
├────────────────────────────────────┼───────────────┼─────────────────────────────────────────────┤
│ Baselines in saved metrics         │ ❓ Unverified │ reports/metrics.json doesn't exist          │
├────────────────────────────────────┼───────────────┼─────────────────────────────────────────────┤
│ Fitted transformers see only train │ ✅ Pass       │ fit/ predict pattern correct                │
└────────────────────────────────────┴───────────────┴─────────────────────────────────────────────┘

The repo is well-structured. The only gaps are: (1) no assertion in code to bound row loss (relies on tests), and (2) cannot verify metrics.json contents since it hasn't been generated yet.

## Verdict
It seems that the majority of the repository is well structured. Claude's assessment regarding forbidden columns seems correct. The assessment on lag seems correct as well, adding the dates prior to calculating the lag. I feel that Claude's assessment regarding row loss is a bit overly cautious, given the fixed tests. The main thing that Claude flagged seems to be the lack of metrics, which prevented me from being able to see what scores were outputted.