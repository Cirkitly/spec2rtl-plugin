# Phase Context Template

Template for `.planning/phases/XX-name/{phase}-CONTEXT.md` - captures implementation decisions for a phase.

**Purpose:** Document decisions that downstream agents need. Researcher uses this to know WHAT to investigate. Planner uses this to know WHAT choices are locked vs flexible.

**Key principle:** Categories are NOT predefined. They emerge from what was actually discussed.

**Downstream consumers:**
- `s2r-phase-researcher` — Reads decisions to focus research (e.g., "AXI4 Interface" → research AXI spec)
- `s2r-planner` — Reads decisions to create specific tasks (e.g., "Async Reset" → ensure synchronizers in plan)

---

## File Template

```markdown
# Phase [X]: [Name] - Context

**Gathered:** [date]
**Status:** Ready for planning

<domain>
## Phase Boundary

[Clear statement of what this phase delivers — the scope anchor. This comes from ROADMAP.md and is fixed. Discussion clarifies implementation within this boundary.]

</domain>

<decisions>
## Implementation Decisions

### [Area 1: e.g., Micro-Architecture]
- [Specific decision made]
- [Another decision if applicable]

### [Area 2: e.g., Interfaces/Protocols]
- [Specific decision made]

### [Area 3: e.g., PPA Constraints]
- [Specific decision made]

### Claude's Discretion
[Areas where user explicitly said "you decide" — Claude has flexibility here during planning/implementation]

</decisions>

<specifics>
## Specific Ideas

[Any particular references, examples, or "I want it like X" moments from discussion. Datasheet references, timing diagrams, specific behaviors.]

[If none: "No specific requirements — open to standard approaches"]

</specifics>

<deferred>
## Deferred Ideas

[Ideas that came up during discussion but belong in other phases. Captured here so they're not lost, but explicitly out of scope for this phase.]

[If none: "None — discussion stayed within phase scope"]

</deferred>

---

*Phase: XX-name*
*Context gathered: [date]*
```

<good_examples>

**Example 1: DDR4 Memory Controller**

```markdown
# Phase 2: Memory Controller - Context

**Gathered:** 2026-02-20
**Status:** Ready for planning

<domain>
## Phase Boundary
Implement DDR4 Controller logic including Initialization, Refresh, and Read/Write leveling. PHY interface is DFI 4.0.
</domain>

<decisions>
## Implementation Decisions

### Protocols
- User Interface: AXI4 (Full), 128-bit width
- PHY Interface: DFI 4.0, 1:4 frequency ratio

### Micro-Architecture
- Reordering buffer depth: 16 entries
- Look-ahead bank management
- Open-Page policy preferred for sequential access

### Clocking
- Controller Clock: 200 MHz
- PHY Clock: 800 MHz (1600 MT/s)
- CDC required on AXI-to-Controller path

### Claude's Discretion
- Refresh scheduler algorithm (Round robin vs Bursts)
- Exact state machine encoding
- Scrambling polynomial

</decisions>

<specifics>
## Specific Ideas
- "Ensure we satisfy the tRFC parameter from the Micron datasheet"
- "Use the open-source LiteDRAM as a reference for the initialization sequence"
</specifics>

<deferred>
## Deferred Ideas
- ECC support — Phase 4
- Power down modes — Post-MVP
</deferred>
```

**Example 2: UART Peripheral**

```markdown
# Phase 1: UART - Context

**Gathered:** 2026-02-20
**Status:** Ready for planning

<domain>
## Phase Boundary
Simple UART with TX/RX and adjustable baud rate. APB interface.
</domain>

<decisions>
## Implementation Decisions

### Interface
- APB3 Slave Interface
- Interrupt output on RX Valid and TX Done

### FIFO
- 16-byte TX FIFO, 16-byte RX FIFO
- Configurable triggers (1/4, 1/2, 3/4, Full)

### Error Handling
- Frame Error detection required
- Parity Error detection required (Odd/Even/None support)

### Claude's Discretion
- Oversampling rate (8x vs 16x)
- Baud rate generator implementation (Fractional vs Integer)

</decisions>
```

</good_examples>

<guidelines>
**This template captures DECISIONS for downstream agents.**

The output should answer: "What does the researcher need to investigate? What choices are locked for the planner?"

**Good content (concrete decisions):**
- "AXI4, 128-bit"
- "FIFO depth 16"
- "Async reset, active low"

**Bad content (too vague):**
- "Fast memory access"
- "Standard interface"
- "Good error handling"

**After creation:**
- File lives in phase directory: `.planning/phases/XX-name/{phase}-CONTEXT.md`
- `s2r-phase-researcher` uses decisions to focus investigation
- `s2r-planner` uses decisions + research to create executable tasks
</guidelines>
