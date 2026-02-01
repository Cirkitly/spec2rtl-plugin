# Phase Prompt Template

> **Note:** Planning methodology is in `spec2rtl/agents/s2r-planner.md`.
> This template defines the PLAN.md output format that the agent produces.

Template for `.planning/phases/XX-name/{phase}-{plan}-PLAN.md` - executable phase plans optimized for parallel execution.

**Naming:** Use `{phase}-{plan}-PLAN.md` format (e.g., `01-02-PLAN.md` for Phase 1, Plan 2)

---

## File Template

```markdown
---
phase: XX-name
plan: NN
type: execute                   # 'execute' (standard) or 'vdd' (Verification Driven Development)
wave: N                         # Execution wave (1, 2, 3...). Pre-computed at plan time.
depends_on: []                  # Plan IDs this plan requires (e.g., ["01-01"]).
files_modified: []              # Files this plan modifies.
autonomous: true                # false if plan has checkpoints requiring user interaction
user_setup: []                  # Human-required setup Claude cannot automate (e.g., Licensing)

# Goal-backward verification (derived during planning, verified after execution)
must_haves:
  truths: []                    # Observable behaviors that must be true (e.g., "ready_o asserts after valid_i")
  artifacts: []                 # Files that must exist with real implementation
  key_links: []                 # Critical connections between modules
---

<objective>
[What this plan accomplishes]

Purpose: [Why this matters for the project]
Output: [What artifacts will be created - RTL, Testbench, Constraints]
</objective>

<execution_context>
@~/.claude/spec2rtl/workflows/execute-plan.md
@~/.claude/spec2rtl/templates/summary.md
[If plan contains checkpoint tasks (type="checkpoint:*"), add:]
@~/.claude/spec2rtl/references/checkpoints.md
</execution_context>

<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md

# Only reference prior plan SUMMARYs if genuinely needed:
# - This plan uses types/parameters from prior plan
# - Prior plan made decision that affects this plan
# Do NOT reflexively chain: Plan 02 refs 01, Plan 03 refs 02...

[Relevant source files:]
@rtl/path/to/relevant.sv
</context>

<tasks>

<task type="auto">
  <name>Task 1: [Action-oriented name]</name>
  <files>rtl/file.sv, tb/tb_file.sv</files>
  <action>[Specific implementation - what to do, how to do it, what to avoid and WHY]</action>
  <verify>[Simulation command or Lint check to prove it worked]</verify>
  <done>[Measurable acceptance criteria]</done>
</task>

<task type="auto" vdd="true">
  <name>Task 2: [Feature Name] (VDD)</name>
  <files>rtl/module.sv, tb/tb_module.sv</files>
  <action>Implement [feature] using VDD cycle (Assert -> Implement -> Verify).</action>
  <verify>Simulation passes with full coverage</verify>
  <done>Assertions pass, RTL implements logic</done>
</task>

<!-- For checkpoint task examples and patterns, see @~/.claude/spec2rtl/references/checkpoints.md -->

<task type="checkpoint:decision" gate="blocking">
  <decision>[What needs deciding]</decision>
  <context>[Why this decision matters - PPA/Complexity]</context>
  <options>
    <option id="option-a"><name>[Name]</name><pros>[Benefits]</pros><cons>[Tradeoffs]</cons></option>
    <option id="option-b"><name>[Name]</name><pros>[Benefits]</pros><cons>[Tradeoffs]</cons></option>
  </options>
  <resume-signal>Select: option-a or option-b</resume-signal>
</task>

<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[What Claude built] - Waveform at [path/to/dump.vcd]</what-built>
  <how-to-verify>Open waveform viewer. Check: [specific signal transitions].</how-to-verify>
  <resume-signal>Type "approved" or describe issues</resume-signal>
</task>

</tasks>

<verification>
Before declaring plan complete:
- [ ] [Specific simulation command]
- [ ] [Linting passes]
- [ ] [Timing check (if synthesis involved)]
</verification>

<success_criteria>
- All tasks completed
- All verification checks pass
- No X-propagation or Latch inferences
- [Plan-specific criteria]
</success_criteria>

<output>
After completion, create `.planning/phases/XX-name/{phase}-{plan}-SUMMARY.md`
</output>
```

---

## Frontmatter Fields

| Field | Required | Purpose |
|-------|----------|---------|
| `phase` | Yes | Phase identifier (e.g., `01-architecture`) |
| `plan` | Yes | Plan number within phase (e.g., `01`, `02`) |
| `type` | Yes | `execute` (standard) or `vdd` (Verification Driven Development) |
| `wave` | Yes | Execution wave number (1, 2, 3...). Pre-computed at plan time. |
| `depends_on` | Yes | Array of plan IDs this plan requires. |
| `files_modified` | Yes | Files this plan touches. |
| `autonomous` | Yes | `true` if no checkpoints, `false` if has checkpoints |
| `user_setup` | No | Array of human-required setup items (Licenses, FPGA Boards) |
| `must_haves` | Yes | Goal-backward verification criteria (see below) |

**Wave is pre-computed:** Wave numbers are assigned during `/s2r:plan-phase`. Execute-phase reads `wave` directly from frontmatter and groups plans by wave number. No runtime dependency analysis needed.

---

## Parallel vs Sequential (Hardware Context)

<parallel_examples>

**Wave 1 candidates (parallel):**

```yaml
# Plan 01 - ALU Implementation
wave: 1
depends_on: []
files_modified: [rtl/alu.sv, tb/tb_alu.sv]
autonomous: true

# Plan 02 - Register File (independent of ALU)
wave: 1
depends_on: []
files_modified: [rtl/regfile.sv, tb/tb_regfile.sv]
autonomous: true
```

**Sequential (genuine dependency):**

```yaml
# Plan 01 - AXI4-Lite Interface
wave: 1
depends_on: []
files_modified: [rtl/axi_slave.sv]

# Plan 02 - Control Registers (needs AXI slave)
wave: 2
depends_on: ["01"]
files_modified: [rtl/csr_bank.sv]
```

**Checkpoint plan:**

```yaml
# Plan 03 - FPGA Synthesis & Test
wave: 3
depends_on: ["01", "02"]
files_modified: [constraints/pins.xdc]
autonomous: false  # Has checkpoint:human-verify (Bitstream load)
```

</parallel_examples>

---

## Context Section

**Parallel-aware context:**

```markdown
<context>
@.planning/PROJECT.md
@.planning/ROADMAP.md
@.planning/STATE.md

# Only include SUMMARY refs if genuinely needed:
# - This plan instantiates module from prior plan
# - This plan uses parameters/typedefs from prior plan
#
# Independent plans need NO prior SUMMARY references.

@rtl/relevant/source.sv
</context>
```

---

## Scope Guidance

**Plan sizing:**
- 2-3 tasks per plan
- ~50% context usage maximum
- Hardware: Split by Module or Interface

**When to split:**
- Different Modules (ALU vs Decoder)
- Testbench vs RTL (sometimes useful to split if large)
- >3 tasks

**Vertical slices preferred:**

```
PREFER: Plan 01 = UART TX (RTL + TB)
        Plan 02 = UART RX (RTL + TB)

AVOID:  Plan 01 = All RTL (TX + RX)
        Plan 02 = All TB (TX + RX)
```

---

## VDD Plans (Verification Driven Development)

Hardware features usually use `type: vdd` unless trivial.

**Heuristic:** Can you write an Assertion (SVA) or Self-Checking Testbench before the RTL?
→ Yes: Create a VDD plan
→ No: Standard task (e.g., boilerplate, wiring)

See `~/.claude/spec2rtl/references/vdd.md` for VDD plan structure.

---

## Task Types

| Type | Use For | Autonomy |
|------|---------|----------|
| `auto` | RTL/TB Implementation, Scripting | Fully autonomous |
| `checkpoint:human-verify` | Waveform inspection, FPGA LED check | Pauses, returns to orchestrator |
| `checkpoint:decision` | Architectural choices (FIFO depth, Arbiter type) | Pauses, returns to orchestrator |
| `checkpoint:human-action` | Source licenses, Plug in board | Pauses, returns to orchestrator |

---

## Must-Haves (Goal-Backward Verification)

The `must_haves` field defines what must be TRUE for the phase goal to be achieved.

**Structure:**

```yaml
must_haves:
  truths:
    - "Ready signal drops when FIFO is full"
    - "Data out matches Data in after latency"
    - "Reset clears all internal state"
  artifacts:
    - path: "rtl/fifo.sv"
      provides: "FIFO Logic"
      min_lines: 50
    - path: "tb/tb_fifo.sv"
      provides: "Self-checking testbench"
      contains: "assert property"
  key_links:
    - from: "rtl/fifo.sv"
      to: "rtl/top.sv"
      via: "Instantiation"
      pattern: "fifo_inst"
```

**Verification flow:**
1. Plan-phase derives must_haves
2. Execute-phase runs plans
3. Verification subagent checks code/simulation logs against must_haves
4. Gaps -> Fix Plans
