# Seeded Faults

## Exercise 1: cnt_change_1 - Failing test 
```text
E           KeyError: 'cnt_change_1'
=========================== short test summary info ============================
FAILED tests/test_leakage.py::test_cnt_change_1_computed_correctly - KeyError...
1 failed, 14 passed in 70.76s (0:01:10)
```

## Fault 1
Added: `or c == "casual"` to `feature_columns`

```text
PostToolUse:Edit hook reported a problem
[${CLAUDE_PROJECT_DIR}/.claude/hooks/check_leakage.sh]: Leakage tests failed after editing /mnt/c/Users/jwang/OneDrive/Documents/GitHub/dsc180a-methodology-6/bikeshare-open/src/features.py:

E       AssertionError: forbidden columns in feature set: {'casual'}

======================================= short test summary info =======================================
FAILED tests/test_leakage.py::test_no_forbidden_features - AssertionError: forbidden columns in feat...
1 failed, 7 passed in 35.29s
```

Repair took 1 Claude turn.

## Fault 2
Removed `.shift(1)` from `cnt_roll_7` in src_feature.py. Rolling mean is included in forecast day's ride count.

**Test output**
```text
E           assert np.float64(4389.0) == 4414.857142857143 ± 0.00441486
E
E             comparison failed
E             Obtained: 4389.0
E             Expected: 4414.857142857143 ± 0.00441486

tests/test_leakage.py:57: AssertionError
======================================= short test summary info =======================================
FAILED tests/test_leakage.py::test_rolling_mean_excludes_today - assert np.float64(4389.0) == 4414.857142857143 ± 0.00441486
1 failed in 7.47s
```

**Hook replay**
```text
Leakage tests failed after editing src/features.py:
E             Obtained: 4389.0
E             Expected: 4414.857142857143 ± 0.00441486
tests/test_leakage.py:57: AssertionError
FAILED tests/test_leakage.py::test_rolling_mean_excludes_today - assert np.fl...
1 failed, 7 passed in 9.76s
```

Repair took 1 Claude turn.

## Fault 3
In `src/features.py`, replaced the `raise ValueError(...)` in `encode_weather` with `pass`, so an unknown weather label no longer raised an error.

**Hook replay**
```text
Leakage tests failed after editing src/features.py:
E       Failed: DID NOT RAISE ValueError
tests/test_leakage.py:78: Failed
FAILED tests/test_leakage.py::test_encode_weather_raises_on_unseen - Failed: ...
1 failed, 8 passed in 7.39s
```

Repair took 1 Claude turn.
