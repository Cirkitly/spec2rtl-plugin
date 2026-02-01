# Model Profiles

Model profiles control which Claude model each Spec2RTL agent uses. This allows balancing quality vs token spend.

## Profile Definitions

| Agent | `quality` | `balanced` | `budget` |
|-------|-----------|------------|----------|
| s2r-planner | opus | opus | sonnet |
| s2r-executor | opus | sonnet | sonnet |
| s2r-uvm-engineer | opus | sonnet | sonnet |
| s2r-codebase-mapper | sonnet | haiku | haiku |
| general-purpose | sonnet | sonnet | haiku |

## Profile Philosophy

**quality** - Maximum reasoning power
- Opus for all decision-making and complex RTL/UVM generation.
- Use when: Architecture definition, Complex FSM design, difficult timing closure.

**balanced** (default) - Smart allocation
- Opus for **Planning** (Architecture choices are expensive to fix later).
- Sonnet for **Execution** (RTL coding follows the plan).
- Sonnet for **Verification** (UVM/SVA analysis).
- Haiku for **Mapping** (File discovery).

**budget** - Minimal Opus usage
- Sonnet for everything.
- Haiku for read-only tasks.
- Use when: conserving quota, routine lint fixes, regression running.

## Design Rationale

**Why Opus for s2r-planner?**
Planning involves micro-architecture trade-offs (Area vs Speed, Bus Protocol selection). These decisions ripple through the entire project.

**Why Sonnet for s2r-uvm-engineer?**
Verification requires understanding the spec and finding gaps. Sonnet is excellent at "Goal-Backward" analysis ("What implies this requirement is met?").

**Why Haiku for s2r-codebase-mapper?**
This agent mostly lists files, checks for "module" keywords, and maps directory structures. It is a read-only pattern matching task.
