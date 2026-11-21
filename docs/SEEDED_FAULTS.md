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

Repair took 1 Claude turn
