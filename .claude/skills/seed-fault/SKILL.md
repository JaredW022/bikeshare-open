---
name: seed-fault
description: Introduce one deliberate bikeshare code fault and capture the test and hook evidence.
---

## Fault to demonstrate

$ARGUMENTS

1. Identify the exact edit and the test expected to catch the fault.
2. Apply only that edit using Claude Code’s Edit tool.
3. Append a new section at the end of `docs/SEEDED_FAULTS.md`.
   Include the exact edit and actual failing test output. Paste the
   automatic hook output if it appeared. Otherwise mark it unverified
   and label any manual replay separately.
4. Show the source and notes diffs. Confirm the notes diff is
   append-only, then wait for my approval before committing.
5. When I ask for the repair, reverse only the fault and run `make test`.
   Report the result so I can record how many repair turns it took.