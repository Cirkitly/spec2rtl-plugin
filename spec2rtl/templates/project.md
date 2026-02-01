# [Project Name]

## What This Is

[Current accurate description — 2-3 sentences. What IP, SoC, or Verification environment is this?
Use standard hardware terminology (e.g., AXI Bridge, RISC-V Core, UVM TB).]

## PPA Targets (Core Value)

[The primary Power, Performance, and Area goals.
e.g., "Must run at 1GHz on 7nm", "Leakage under 5mW", "Area < 0.5mm²", "Latency < 5 cycles"]

## Hardware Specifications

### Interfaces & Protocols
- [ ] [Interface 1] (e.g., AXI4-Stream Slave, 32-bit)
- [ ] [Interface 2] (e.g., APB Control Interface)
- [ ] [Interface 3] (e.g., Interrupts, GPIOs)

### Clocking & Reset
- [ ] [Clock Domain 1] (Freq, Jitter reqs)
- [ ] [Reset Strategy] (Async active low, Sync release)

## Verification Strategy

### Validated
(None yet — pass regressions to validate)

### Active Requirements (RTL & TB)
- [ ] [Requirement 1]
- [ ] [Requirement 2]

### Out of Scope
- [Exclusion 1] — [why]

## Context

[Background information:
- Target Technology Node (e.g., TSMC 7nm, Xilinx Ultrascale+)
- EDA Tools Version (e.g., Vivado 2023.2, VCS 2024)
- Legacy IP or VIP dependencies]

## Constraints

- **[Type]**: [What] — [Why]
- **[Type]**: [What] — [Why]

Common types: Tech Node, Die Size, Power Budget, Latency (cycles), Throughput.

## Key Decisions

<!-- Architectural decisions (e.g., FIFO depth, arbitration scheme). -->

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| [Choice] | [Why] | [✓ Good / ⚠️ Revisit / — Pending] |

---
*Last updated: [date] after [trigger]*
