# Verification Report Template

Template for `.planning/phases/XX-name/{phase}-VERIFICATION.md` — phase goal verification results.

---

## File Template

```markdown
---
phase: XX-name
verified: YYYY-MM-DDTHH:MM:SSZ
status: passed | gaps_found | human_needed
score: N/M must-haves verified
---

# Phase {X}: {Name} Verification Report

**Phase Goal:** {goal from ROADMAP.md}
**Verified:** {timestamp}
**Status:** {passed | gaps_found | human_needed}

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | {truth from must_haves} | ✓ VERIFIED | {what confirmed it} |
| 2 | {truth from must_haves} | ✗ FAILED | {what's wrong} |
| 3 | {truth from must_haves} | ? UNCERTAIN | {why can't verify} |

**Score:** {N}/{M} truths verified

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `rtl/alu.sv` | ALU Logic | ✓ EXISTS + SUBSTANTIVE | Module exists, implements ops, >50 lines |
| `tb/tb_alu.sv` | Self-checking TB | ✗ STUB | File exists but only contains empty module |
| `constraints/timing.xdc` | Clock definition | ✓ EXISTS + SUBSTANTIVE | create_clock defined for clk |

**Artifacts:** {N}/{M} verified

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|----|--------|---------|
| cpu_core.sv | alu.sv | instantiation | ✓ WIRED | `alu u_alu (.*)` found |
| top.sv | cpu_core.sv | interrupt map | ✗ NOT WIRED | `irq` port unconnected `()` |
| rst_gen.sv | cpu_core.sv | reset tree | ✓ WIRED | `rst_n` connected to `sys_rst_n` |

**Wiring:** {N}/{M} connections verified

## Requirements Coverage

| Requirement | Status | Blocking Issue |
|-------------|--------|----------------|
| {REQ-01}: {description} | ✓ SATISFIED | - |
| {REQ-02}: {description} | ✗ BLOCKED | TB fails for this feature |
| {REQ-03}: {description} | ? NEEDS HUMAN | Verify LED blink rate on board |

**Coverage:** {N}/{M} requirements satisfied

## Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| rtl/decoder.sv | 12 | `always @(a)` (Latch) | 🛑 Blocker | Timing unpredictable |
| rtl/fifo.sv | 45 | `// TODO: Empty logic` | 🛑 Blocker | Logic incomplete |
| rtl/uart.sv | - | Missing timescale | ⚠️ Warning | Simulation time precision issues |

**Anti-patterns:** {N} found ({blockers} blockers, {warnings} warnings)

## Human Verification Required

{If no human verification needed:}
None — all verifiable items checked programmatically.

{If human verification needed:}

### 1. {Test Name}
**Test:** {What to do - e.g., Check Waveform}
**Expected:** {What should happen - e.g., Valid asserts 1 cycle after Ready}
**Why human:** {Why can't verify programmatically}

### 2. {Test Name}
**Test:** {What to do - e.g., Board LED Check}
**Expected:** {What should happen}
**Why human:** {Physical interaction required}

## Gaps Summary

{If no gaps:}
**No gaps found.** Phase goal achieved. Ready to proceed.

{If gaps found:}

### Critical Gaps (Block Progress)

1. **{Gap name}**
   - Missing: {what's missing}
   - Impact: {why this blocks the goal}
   - Fix: {what needs to happen}

2. **{Gap name}**
   - Missing: {what's missing}
   - Impact: {why this blocks the goal}
   - Fix: {what needs to happen}

### Non-Critical Gaps (Can Defer)

1. **{Gap name}**
   - Issue: {what's wrong}
   - Impact: {limited impact because...}
   - Recommendation: {fix now or defer}

## Recommended Fix Plans

{If gaps found, generate fix plan recommendations:}

### {phase}-{next}-PLAN.md: {Fix Name}

**Objective:** {What this fixes}

**Tasks:**
1. {Task to fix gap 1}
2. {Task to fix gap 2}
3. {Verification task}

**Estimated scope:** {Small / Medium}

---

### {phase}-{next+1}-PLAN.md: {Fix Name}

**Objective:** {What this fixes}

**Tasks:**
1. {Task}
2. {Task}

**Estimated scope:** {Small / Medium}

---

## Verification Metadata

**Verification approach:** Goal-backward (derived from phase goal)
**Must-haves source:** {PLAN.md frontmatter | derived from ROADMAP.md goal}
**Automated checks:** {N} passed, {M} failed
**Human checks required:** {N}
**Total verification time:** {duration}

---
*Verified: {timestamp}*
*Verifier: Claude (subagent)*
```

---

## Guidelines

**Status values:**
- `passed` — All must-haves verified, no blockers
- `gaps_found` — One or more critical gaps found
- `human_needed` — Automated checks pass but human verification required

**Evidence types:**
- For EXISTS: "File at path, module definition found"
- For SUBSTANTIVE: "N lines, has patterns X, Y, Z"
- For WIRED: "Line N: instantiation found with port map"
- For FAILED: "Missing because X" or "Stub because Y"

**Severity levels:**
- 🛑 Blocker: Prevents goal achievement (Latch, X-prop, Syntax Error)
- ⚠️ Warning: Indicates incomplete but doesn't block (Missing Timescale, TODO)
- ℹ️ Info: Notable but not problematic

**Fix plan generation:**
- Only generate if gaps_found
- Group related fixes into single plans
- Keep to 2-3 tasks per plan
- Include verification task in each plan

---

## Example

```markdown
---
phase: 02-mem-ctrl
verified: 2026-02-20T14:30:00Z
status: gaps_found
score: 2/5 must-haves verified
---

# Phase 2: Memory Controller Verification Report

**Phase Goal:** Working DDR4 Controller with Initialization and Refresh logic
**Verified:** 2026-02-20T14:30:00Z
**Status:** gaps_found

## Goal Achievement

### Observable Truths

| # | Truth | Status | Evidence |
|---|-------|--------|----------|
| 1 | Init sequence completes | ✗ FAILED | Simulation hangs at state INIT_CALIB |
| 2 | Refresh interval met | ? UNCERTAIN | Need waveform inspection of tREFI |
| 3 | Read/Write operations work | ✗ FAILED | Blocked by Init failure |
| 4 | Controller issues DFI commands | ✓ VERIFIED | DFI bus toggles correctly in logs |
| 5 | No X-prop on reset | ✓ VERIFIED | Reset assertion clears all regs |

**Score:** 2/5 truths verified

### Required Artifacts

| Artifact | Expected | Status | Details |
|----------|----------|--------|---------|
| `rtl/ddr_ctrl.sv` | Controller Top | ✓ EXISTS + SUBSTANTIVE | Exports ddr_ctrl, 500 lines |
| `rtl/phy_if.sv` | DFI Interface | ✓ EXISTS + SUBSTANTIVE | Implements DFI 4.0 protocol |
| `tb/tb_ddr_ctrl.sv` | Testbench | ✓ EXISTS + SUBSTANTIVE | Instantiates DUT and VIP |
| `rtl/refresh_gen.sv` | Refresh Logic | ✗ STUB | File empty |

**Artifacts:** 3/4 verified

### Key Link Verification

| From | To | Via | Status | Details |
|------|----|----|--------|---------|
| ddr_ctrl.sv | phy_if.sv | instantiation | ✓ WIRED | `phy_if u_phy` connected |
| ddr_ctrl.sv | refresh_gen.sv | instantiation | ✗ NOT WIRED | Module not instantiated |
| top.sv | ddr_ctrl.sv | axi_bus | ✓ WIRED | AXI ports mapped |

**Wiring:** 2/3 connections verified

## Requirements Coverage

| Requirement | Status | Blocking Issue |
|-------------|--------|----------------|
| MEM-01: DFI 4.0 Compliant | ✓ SATISFIED | - |
| MEM-02: Auto-Refresh | ✗ BLOCKED | Refresh logic missing |
| MEM-03: AXI4 Slave | ✓ SATISFIED | - |

**Coverage:** 2/3 requirements satisfied

## Anti-Patterns Found

| File | Line | Pattern | Severity | Impact |
|------|------|---------|----------|--------|
| rtl/phy_if.sv | 120 | `always @(posedge clk)` (No Reset) | ⚠️ Warning | Potential X state |
| rtl/ddr_ctrl.sv | - | `// TODO: Implement Arbiter` | 🛑 Blocker | Core logic missing |

**Anti-patterns:** 2 found (1 blocker, 1 warning)

## Human Verification Required

### 1. Check Refresh Interval
**Test:** Open `dump.vcd`
**Expected:** `dfi_cs_n` asserts periodically every 7.8us
**Why human:** Simulation didn't run long enough to verify automatically

## Gaps Summary

### Critical Gaps (Block Progress)

1. **Refresh Logic Missing**
   - Missing: `refresh_gen.sv` implementation
   - Impact: Controller violates DRAM spec, data loss
   - Fix: Implement refresh counter and FSM

2. **Initialization Hang**
   - Issue: Stuck in Calibration state
   - Impact: Controller never becomes ready
   - Fix: Debug `phy_init_done` signal path

## Recommended Fix Plans

### 02-04-PLAN.md: Fix Refresh and Init

**Objective:** Implement missing refresh logic and debug init hang

**Tasks:**
1. Implement `rtl/refresh_gen.sv`
2. Instantiate and wire in `ddr_ctrl.sv`
3. Debug `phy_init_done` logic in `rtl/phy_if.sv`
4. Verify: Simulation completes init and sends refreshes

**Estimated scope:** Medium
```
