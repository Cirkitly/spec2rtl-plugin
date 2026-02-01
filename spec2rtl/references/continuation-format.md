# Continuation Format

Standard format for presenting next steps after completing a command or workflow.

## Core Structure

```
---

## ▶ Next Up

**{identifier}: {name}** — {one-line description}

`{command to copy-paste}`

<sub>`/clear` first → fresh context window</sub>

---

**Also available:**
- `{alternative option 1}` — description
- `{alternative option 2}` — description

---
```

## Format Rules

1. **Always show what it is** — name + description, never just a command path
2. **Pull context from source** — ROADMAP.md for phases, PLAN.md `<objective>` for plans
3. **Command in inline code** — backticks, easy to copy-paste, renders as clickable link
4. **`/clear` explanation** — always include, keeps it concise but explains why
5. **"Also available" not "Other options"** — sounds more app-like
6. **Visual separators** — `---` above and below to make it stand out

## Variants

### Execute Next Plan

```
---

## ▶ Next Up

**02-03: DDR4 Controller** — Implement Calibration FSM and PHY Interface

`/s2r:execute-phase 2`

<sub>`/clear` first → fresh context window</sub>

---

**Also available:**
- Review plan before executing
- `/s2r:list-phase-assumptions 2` — check assumptions
- `/s2r:verify-work 2` — run manual verification

---
```

### Execute Final Plan in Phase

Add note that this is the last plan and what comes after:

```
---

## ▶ Next Up

**02-03: DDR4 Controller** — Implement Calibration FSM and PHY Interface
<sub>Final plan in Phase 2</sub>

`/s2r:execute-phase 2`

<sub>`/clear` first → fresh context window</sub>

---

**After this completes:**
- Phase 2 → Phase 3 transition
- Next: **Phase 3: Peripherals** — UART, SPI, and I2C integration

---
```

### Plan a Phase

```
---

## ▶ Next Up

**Phase 2: Memory Subsystem** — DDR4 Controller and Cache Coherency

`/s2r:plan-phase 2`

<sub>`/clear` first → fresh context window</sub>

---

**Also available:**
- `/s2r:discuss-phase 2` — gather architecture specs first
- `/s2r:research-phase 2` — investigate IP and PHY requirements
- Review roadmap

---
```

### Phase Complete, Ready for Next

Show completion status before next action:

```
---

## ✓ Phase 2 Complete

3/3 plans executed

## ▶ Next Up

**Phase 3: Peripherals** — UART, SPI, and I2C integration

`/s2r:plan-phase 3`

<sub>`/clear` first → fresh context window</sub>

---

**Also available:**
- `/s2r:discuss-phase 3` — gather architecture specs first
- `/s2r:research-phase 3` — investigate IP requirements
- Review what Phase 2 built

---
```

### Multiple Equal Options

When there's no clear primary action:

```
---

## ▶ Next Up

**Phase 3: Peripherals** — UART, SPI, and I2C integration

**To plan directly:** `/s2r:plan-phase 3`

**To discuss context first:** `/s2r:discuss-phase 3`

**To research unknowns:** `/s2r:research-phase 3`

<sub>`/clear` first → fresh context window</sub>

---
```

### Milestone Complete

```
---

## 🎉 Milestone v1.0 Tape-out Complete

All 4 phases verified

## ▶ Next Up

**Start v1.1 Respin** — questioning → research → requirements → roadmap

`/s2r:new-milestone`

<sub>`/clear` first → fresh context window</sub>

---
```

## Pulling Context

### For phases (from ROADMAP.md):

```markdown
### Phase 2: Memory Subsystem
**Goal**: DDR4 Controller and Cache Coherency
```

Extract: `**Phase 2: Memory Subsystem** — DDR4 Controller and Cache Coherency`

### For plans (from ROADMAP.md):

```markdown
Plans:
- [ ] 02-03: Implement Calibration FSM
```

Or from PLAN.md `<objective>`:

```xml
<objective>
Implement DDR4 Calibration FSM to handle DQS gating.

Purpose: Ensure reliable data capture at 1600MT/s.
</objective>
```

Extract: `**02-03: DDR4 Calibration** — Implement Calibration FSM to handle DQS gating`

## Anti-Patterns

### Don't: Command-only (no context)

```
## To Continue

Run `/clear`, then paste:
/s2r:execute-phase 2
```

User has no idea what 02-03 is about.

### Don't: Missing /clear explanation

```
`/s2r:plan-phase 3`

Run /clear first.
```

Doesn't explain why. User might skip it.

### Don't: "Other options" language

```
Other options:
- Review roadmap
```

Sounds like an afterthought. Use "Also available:" instead.

### Don't: Fenced code blocks for commands

```
```
/s2r:plan-phase 3
```
```

Fenced blocks inside templates create nesting ambiguity. Use inline backticks instead.
