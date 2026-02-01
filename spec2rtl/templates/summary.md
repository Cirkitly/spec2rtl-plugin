# Summary Template

Template for `.planning/phases/XX-name/{phase}-{plan}-SUMMARY.md` - phase completion documentation.

---

## File Template

```markdown
---
phase: XX-name
plan: YY
subsystem: [primary category: rtl, verification, synthesis, constraints, ip, firmware]
tags: [searchable tech: axi, systemverilog, uvm, vivado, verilator, cdc]

# Dependency graph
requires:
  - phase: [prior phase this depends on]
    provides: [what that phase built that this uses]
provides:
  - [bullet list of what this phase built/delivered]
affects: [list of phase names or keywords that will need this context]

# Tech tracking
tech-stack:
  added: [IPs/Tools added in this phase]
  patterns: [architectural/code patterns established]

key-files:
  created: [important files created]
  modified: [important files modified]

key-decisions:
  - "Decision 1"
  - "Decision 2"

patterns-established:
  - "Pattern 1: description"
  - "Pattern 2: description"

# Metrics
duration: Xmin
completed: YYYY-MM-DD
# Hardware Metrics (Optional)
area: [LUTs/FFs count if known]
fmax: [Max Frequency if known]
coverage: [Verification coverage %]
---

# Phase [X]: [Name] Summary

**[Substantive one-liner describing outcome - e.g., "AXI4-Lite Slave with 16-entry FIFO and Full Verification"]**

## Performance

- **Duration:** [time]
- **Started:** [ISO timestamp]
- **Completed:** [ISO timestamp]
- **Tasks:** [count completed]
- **Files modified:** [count]

## Accomplishments
- [Most important outcome]
- [Second key accomplishment]
- [Third if applicable]

## Task Commits

Each task was committed atomically:

1. **Task 1: [task name]** - `abc123f` (feat/fix/test/refactor)
2. **Task 2: [task name]** - `def456g` (feat/fix/test/refactor)

**Plan metadata:** `lmn012o` (docs: complete plan)

## Files Created/Modified
- `rtl/module.sv` - What it does
- `tb/tb_module.sv` - What it does

## Decisions Made
[Key decisions with brief rationale, or "None - followed plan as specified"]

## Deviations from Plan

[If no deviations: "None - plan executed exactly as written"]

[If deviations occurred:]

### Auto-fixed Issues

**1. [Rule X - Category] Brief description**
- **Found during:** Task [N]
- **Issue:** [What was wrong]
- **Fix:** [What was done]
- **Verification:** [How it was verified]
- **Committed in:** [hash]

---

**Total deviations:** [N] auto-fixed
**Impact on plan:** [Brief assessment]

## Issues Encountered
[Problems and how they were resolved, or "None"]

## User Setup Required

[If USER-SETUP.md was generated:]
**External services require manual configuration.** See [{phase}-USER-SETUP.md](./{phase}-USER-SETUP.md) for:
- Licenses to source
- Board setup steps

[If no USER-SETUP.md:]
None.

## Next Phase Readiness
[What's ready for next phase]
[Any blockers or concerns]

---
*Phase: XX-name*
*Completed: [date]*
```

<frontmatter_guidance>
**Purpose:** Enable automatic context assembly via dependency graph.

**Subsystem:** rtl, verification, synthesis, constraints, ip, firmware.

**Metrics:**
- `area`: Estimate if synthesis run (e.g., "500 LUTs")
- `fmax`: Timing result if available (e.g., "250 MHz")
- `coverage`: Simulation coverage (e.g., "95%")

</frontmatter_guidance>

<one_liner_rules>
The one-liner MUST be substantive:

**Good:**
- "AXI4-Lite Slave with 16-entry FIFO and Full Verification"
- "DDR4 Controller Initialized and Calibrated in Simulation"
- "Timing Closure achieved at 400MHz for Core Logic"

**Bad:**
- "Phase complete"
- "RTL written"
- "All tasks done"
</one_liner_rules>

<example>
```markdown
# Phase 1: UART Summary

**APB3 UART with 16-byte FIFO and Baud Rate Generator, Verified with Self-Checking TB**

## Performance
- **Duration:** 45 min
- **Coverage:** 100% Line, 95% Toggle

## Accomplishments
- Implemented `uart_tx.sv` and `uart_rx.sv`
- Implemented `baud_gen.sv` with fractional support
- Verified loopback in `tb_uart_top.sv`

## Decisions Made
- Used 16x oversampling for RX robustness
- Selected Odd Parity as default

## Deviations
### Auto-fixed Issues
**1. [Rule 2 - Missing Critical] Added 2-stage synchronizer for RX input**
- **Found during:** Task 1 (RX Implementation)
- **Issue:** RX input is asynchronous, potential metastability
- **Fix:** Added `sync_rx.sv` module instantiation
```
</example>
