# Milestone Entry Template

Add this entry to `.planning/MILESTONES.md` when completing a milestone:

```markdown
## v[X.Y] [Name] (Shipped: YYYY-MM-DD)

**Delivered:** [One sentence describing what shipped - e.g., Tape-out GDSII or FPGA Bitstream]

**Phases completed:** [X-Y] ([Z] plans total)

**Key accomplishments:**
- [Major achievement 1]
- [Major achievement 2]
- [Major achievement 3]

**Stats:**
- [X] files created/modified
- [Y] lines of code (SystemVerilog/VHDL)
- [Z] phases, [N] plans, [M] tasks
- [D] days from start to ship

**Git range:** `feat(XX-XX)` → `feat(YY-YY)`

**What's next:** [Brief description of next milestone goals, or "Project complete"]

---
```

<structure>
If MILESTONES.md doesn't exist, create it with header:

```markdown
# Project Milestones: [Project Name]

[Entries in reverse chronological order - newest first]
```
</structure>

<guidelines>
**When to create milestones:**
- Tape-out (GDSII delivery)
- FPGA Release (Bitstream generation)
- Major Verification Milestone (100% Coverage)
- Before archiving planning (capture what was shipped)

**Stats to include:**
- Count modified files: `git diff --stat feat(XX-XX)..feat(YY-YY) | tail -1`
- Count LOC: `find . -name "*.sv" -o -name "*.v" -o -name "*.vhd" | xargs wc -l`
- Phase/plan/task counts from ROADMAP
- Timeline from first phase commit to last phase commit

**Git range format:**
- First commit of milestone → last commit of milestone
- Example: `feat(01-01)` → `feat(04-01)` for phases 1-4
</guidelines>

<example>
```markdown
# Project Milestones: RISC-V SoC

## v1.0 FPGA Release (Shipped: 2026-03-15)

**Delivered:** Bootable Linux image on Xilinx ZCU102 via SD Card

**Phases completed:** 1-4 (7 plans total)

**Key accomplishments:**
- Implemented RV64GC Core
- Integrated DDR4 Controller
- Validated UART/SPI/I2C Peripherals
- Timing Closure at 100MHz

**Stats:**
- 47 files created
- 12,450 lines of SystemVerilog
- 4 phases, 7 plans, 28 tasks
- 12 days from start to ship

**Git range:** `feat(01-01)` → `feat(04-01)`

**What's next:** Performance optimization and Custom Instruction extension (v1.1)
```
</example>
