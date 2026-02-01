# User Setup Template

Template for `.planning/phases/XX-name/{phase}-USER-SETUP.md` - human-required configuration that Claude cannot automate.

**Purpose:** Document setup tasks that literally require human action - Licenses, Board setup, JTAG drivers. Claude automates everything possible (scripts, simulation); this file captures only what remains.

---

## File Template

```markdown
# Phase {X}: User Setup Required

**Generated:** [YYYY-MM-DD]
**Phase:** {phase-name}
**Status:** Incomplete

Complete these items for the design/verification to proceed. Claude automated everything possible; these items require human access to physical hardware or vendor portals.

## Environment Variables / Licenses

| Status | Variable | Source | Add to |
|--------|----------|--------|--------|
| [ ] | `LM_LICENSE_FILE` | [Vendor Portal → Licensing] | `.bashrc` or Session |
| [ ] | `XILINX_VIVADO` | [Installation Path] | `$PATH` |

## Hardware Setup

[Only if physical hardware is involved]

- [ ] **Connect Board**
  - Board: [Board Name]
  - JTAG Cable: [USB Blaster / Digilent HS2]
  - Power: [On/Off]

- [ ] **Dip Switch Configuration**
  - SW[3:0]: [0011] (Boot mode)
  - Jumpers: [J1 Shorted]

## Tool Configuration

[Only if tool setup requires GUI interaction]

- [ ] **Install Board Files**
  - Location: [Vendor Store / Git Repo]
  - Path: `[Vivado Install]/data/boards/board_files`

## Verification

After completing setup, verify with:

```bash
# [Verification commands]
# e.g.,
vlog -version
djtgcfg enum
```

Expected results:
- [What success looks like]

---

**Once all items complete:** Mark status as "Complete" at top of file.
```

---

## When to Generate

Generate `{phase}-USER-SETUP.md` when plan frontmatter contains `user_setup` field.

**Trigger:** `user_setup` exists in PLAN.md frontmatter and has items.

**Location:** Same directory as PLAN.md and SUMMARY.md.

**Timing:** Generated during execute-plan.md after tasks complete, before SUMMARY.md creation.

---

## Frontmatter Schema

In PLAN.md, `user_setup` declares human-required configuration:

```yaml
user_setup:
  - service: vivado_license
    why: "Synthesis requires valid license"
    env_vars:
      - name: LM_LICENSE_FILE
        source: "Xilinx Licensing Portal"
  - service: fpga_board
    why: "Bitstream loading"
    hardware:
      - "Connect Digilent HS2 JTAG cable"
      - "Set Boot Mode jumper to JTAG"
```

---

## The Automation-First Rule

**USER-SETUP.md contains ONLY what Claude literally cannot do.**

| Claude CAN Do (not in USER-SETUP) | Claude CANNOT Do (→ USER-SETUP) |
|-----------------------------------|--------------------------------|
| `make build` | Download License File from Portal |
| Write TCL scripts | Plug in USB Cable |
| Parse Simulation Logs | Toggle Physical Switches |
| Run `vivado -mode batch` | Install proprietary drivers (sometimes) |
| Generate Bitstream | Look at LEDs on board |

**The test:** "Does this require a human hand or credentials for a vendor portal?"
- Yes → USER-SETUP.md
- No → Claude does it automatically
