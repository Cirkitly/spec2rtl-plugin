# Requirements Template

Template for `.planning/REQUIREMENTS.md` — checkable requirements that define "done."

<template>

```markdown
# Requirements: [Project Name]

**Defined:** [date]
**PPA Targets:** [from PROJECT.md]

## v1 Specifications

Requirements for initial Tape-out / FPGA Image. Each maps to roadmap phases.

### Interfaces & I/O

- [ ] **INT-01**: [Interface Description] (e.g., AXI4-Stream Slave, 32-bit)
- [ ] **INT-02**: [Interface Description] (e.g., APB Control Interface)
- [ ] **INT-03**: [Signal Description] (e.g., Interrupt output, active high)

### Functional Logic

- [ ] **FUNC-01**: [Logic Requirement] (e.g., Pipeline depth < 10 cycles)
- [ ] **FUNC-02**: [Buffer Requirement] (e.g., FIFO depth 64 words)
- [ ] **FUNC-03**: [Feature Requirement] (e.g., Error handling for malformed packets)

### Clocking & Reset

- [ ] **CLK-01**: [Clock Requirement] (e.g., Support 200MHz system clock)
- [ ] **RST-01**: [Reset Requirement] (e.g., Asynchronous active-low reset)

## v2 Specifications

Deferred to future revision. Tracked but not in current roadmap.

### [Category]

- **[CAT]-01**: [Requirement description]

## Out of Scope

Explicitly excluded. Documented to prevent scope creep.

| Feature | Reason |
|---------|--------|
| [Feature] | [Why excluded] |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| INT-01 | Phase 1 | Pending |
| FUNC-01 | Phase 2 | Pending |

**Coverage:**
- v1 requirements: [X] total
- Mapped to phases: [Y]
- Unmapped: [Z] ⚠️

---
*Requirements defined: [date]*
*Last updated: [date] after [trigger]*
```

</template>

<guidelines>

**Requirement Format:**
- ID: `[CATEGORY]-[NUMBER]` (INT-01, FUNC-02, CLK-03)
- Description: Testable hardware behavior or constraint
- Checkbox: Only for v1 requirements

**Categories:**
- Interfaces (AXI, APB, GPIO, PCIe)
- Functional Logic (Data path, Control path, FSMs)
- Clocking & Reset (Domains, PLLs, Reset trees)
- Performance (Latency, Throughput)
- Power (Gating, Domains)

**Traceability:**
- Each requirement maps to a Phase (RTL or Verification phase)
- Status: Pending -> In Progress -> Verified (via Sim/Formal)

</guidelines>
