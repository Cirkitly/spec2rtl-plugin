---
status: testing | complete | diagnosed
phase: XX-name
source: [list of SUMMARY.md files]
started: [ISO timestamp]
updated: [ISO timestamp]
---

# System Validation / Lab Test

**Purpose:** Verify the design works in the real world (FPGA, Emulation, or Post-Synthesis Simulation).

## Current Test
<!-- OVERWRITE each test - shows where we are -->

number: [N]
name: [test name]
expected: |
  [Physical observation or Waveform event]
awaiting: user action / observation

## Tests

### 1. [Test Name: e.g. "Sanity Blink"]
expected: [e.g. "LED0 blinks at 1Hz"]
result: [pending]

### 2. [Test Name: e.g. "UART Echo"]
expected: [e.g. "Type 'a' in terminal, see 'a' echoed back"]
result: pass

### 3. [Test Name: e.g. "DDR Calibration"]
expected: [e.g. "CALIB_DONE goes high, no data errors"]
result: issue
reported: "CALIB_DONE stays low. Stuck in state READ_DQS."
severity: major

...

## Summary

total: [N]
passed: [N]
issues: [N]
pending: [N]
skipped: [N]

## Gaps

<!-- YAML format for plan-phase --gaps consumption -->
- truth: "[expected behavior]"
  status: failed
  reason: "User reported: [verbatim]"
  severity: blocker | major | minor | cosmetic
  test: [N]
  root_cause: ""     # Filled by diagnosis
  artifacts: []      # Filled by diagnosis
  debug_session: ""  # Filled by diagnosis
