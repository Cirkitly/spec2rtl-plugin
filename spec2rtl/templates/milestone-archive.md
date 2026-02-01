# Milestone Archive Template

This template is used by the complete-milestone workflow to create archive files in `.planning/milestones/`.

---

## File Template

# Milestone v{{VERSION}}: {{MILESTONE_NAME}}

**Status:** ✅ SHIPPED {{DATE}}
**Phases:** {{PHASE_START}}-{{PHASE_END}}
**Total Plans:** {{TOTAL_PLANS}}

## Overview

{{MILESTONE_DESCRIPTION}}

## Phases

{{PHASES_SECTION}}

[For each phase in this milestone, include:]

### Phase {{PHASE_NUM}}: {{PHASE_NAME}}

**Goal**: {{PHASE_GOAL}}
**Depends on**: {{DEPENDS_ON}}
**Plans**: {{PLAN_COUNT}} plans

Plans:

- [x] {{PHASE}}-01: {{PLAN_DESCRIPTION}}
- [x] {{PHASE}}-02: {{PLAN_DESCRIPTION}}
      [... all plans ...]

**Details:**
{{PHASE_DETAILS_FROM_ROADMAP}}

**For decimal phases, include (INSERTED) marker:**

### Phase 2.1: Critical Timing Fix (INSERTED)

**Goal**: Fix Hold Violation on DDR Interface
**Depends on**: Phase 2
**Plans**: 1 plan

Plans:

- [x] 02.1-01: Insert delay buffers

**Details:**
{{PHASE_DETAILS_FROM_ROADMAP}}

---

## Milestone Summary

**Decimal Phases:**

- Phase 2.1: Critical Timing Fix (inserted after Phase 2 for urgent fix)

**Key Decisions:**
{{DECISIONS_FROM_PROJECT_STATE}}
[Example:]

- Decision: Use AXI4 instead of AHB (Rationale: High throughput requirement)
- Decision: Sync Reset (Rationale: FPGA recommended practice)

**Issues Resolved:**
{{ISSUES_RESOLVED_DURING_MILESTONE}}
[Example:]

- Fixed CDC bug in UART
- Resolved Timing Violation in ALU

**Issues Deferred:**
{{ISSUES_DEFERRED_TO_LATER}}
[Example:]

- Power Optimization (deferred until v2.0)

**Technical Debt Incurred:**
{{SHORTCUTS_NEEDING_FUTURE_WORK}}
[Example:]

- Hardcoded arbitration weights in Bus Interconnect

---

_For current project status, see .planning/ROADMAP.md_

---

## Usage Guidelines

<guidelines>
**When to create milestone archives:**
- After completing all phases in a milestone (v1.0 Tape-out, v1.1 Respin, etc.)
- Triggered by complete-milestone workflow
- Before planning next milestone work

**How to fill template:**

- Replace {{PLACEHOLDERS}} with actual values
- Extract phase details from ROADMAP.md
- Document decimal phases with (INSERTED) marker
- Include key decisions from PROJECT-STATE.md or SUMMARY files
- List issues resolved vs deferred
- Capture technical debt for future reference

**Archive location:**

- Save to `.planning/milestones/v{VERSION}-{NAME}.md`
- Example: `.planning/milestones/v1.0-tapeout.md`

**After archiving:**

- Update ROADMAP.md to collapse completed milestone in `<details>` tag
- Update PROJECT.md to brownfield format with Current State section
- Continue phase numbering in next milestone (never restart at 01)
  </guidelines>
