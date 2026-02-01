<overview>
Git integration for Hardware Design & Verification (HDV).
</overview>

<core_principle>
**Commit Source, Ignore Artifacts.**

Hardware builds generate massive binaries (`.bit`, `.dcp`, `.vcd`, `.wdb`) and millions of intermediate files. These must NEVER be committed to the repo.
Only commit what is needed to *reproduce* the design: RTL, Testbenches, Constraints, Scripts, and IP configurations.
</core_principle>

<gitignore_strategy>
## EDA Ignore Patterns

Your `.gitignore` must be aggressive.

**Simulation:**
- `*.vcd`, `*.fsdb`, `*.vpd` (Waveforms - huge!)
- `*.wdb`, `*.db` (Coverage/Debug databases)
- `sim/work/`, `xsim.dir/` (Compilation directories)
- `*.log`, `*.jou` (Logs)

**Synthesis/Implementation:**
- `*.dcp` (Checkpoints - huge!)
- `*.bit`, `*.bin`, `*.mcs` (Bitstreams - regenerate them)
- `*.rpt`, `*.pb` (Reports - generated)
- `input_files/`, `output_files/` (Quartus)
- `.Xil/` (Vivado temp)

**Exceptions (Use LFS):**
- Golden reference models (`golden_model.mex`, `reference_image.png`)
- Verification binaries (`firmware.elf` if hard to rebuild)
</gitignore_strategy>

<commit_points>
| Event | Commit? | Why |
|-------|---------|-----|
| **Task completed** | YES | Atomic unit of work (RTL + TB + Constraints) |
| **Plan completed** | YES | Metadata commit (SUMMARY + STATE + ROADMAP) |
| Simulation Pass | NO | Transient result. Commit the code that made it pass. |
| Bitstream Gen | NO | Artifact. Commit the RTL/Constraints. |
| IP Configuration | YES | Commit the `.xci` or `.tcl` script, not the generated output products. |
</commit_points>

<commit_formats>

<format name="task-completion">
## Task Completion

Each task gets its own commit.

```
{type}({phase}-{plan}): {task-name}

- [Key change 1]
- [Key change 2]
```

**Types:**
- `feat`: RTL implementation / New Logic
- `test`: Testbench / Assertions / Verification
- `fix`: Bug fix in RTL or TB
- `refactor`: Optimization (Area/Timing) without logic change
- `chore`: Build scripts / Constraints / IP update

**Hardware Examples:**

```bash
# Feature (RTL)
git add rtl/core/alu.sv
git commit -m "feat(01-02): implement ALU arithmetic ops
- Added ADD, SUB, AND, OR operations
- Implemented overflow detection logic
- Parameterized width (default 32)
"

# Test (Verification)
git add tb/tb_alu.sv
git commit -m "test(01-02): add directed tests for ALU
- Added corner case testing (max_int + 1)
- Verified reset behavior
- Added self-checking assertions
"

# Fix (Timing)
git add constraints/top.xdc
git commit -m "fix(04-01): relax timing on LED outputs
- Changed false_path for status LEDs
- Fixed hold violation on UART reset
"
```
</format>

<format name="plan-completion">
## Plan Completion

Metadata commit after all tasks are done.

```
docs({phase}-{plan}): complete [plan-name] plan

Tasks completed: [N]/[N]
- Implemented ALU
- Verified Arithmetic Ops
- Closed Timing

SUMMARY: .planning/phases/XX-name/{phase}-{plan}-SUMMARY.md
```
</format>

</commit_formats>

<anti_patterns>
**DO NOT COMMIT:**
- `generated/` directories from IP catalogs (commit the `.xci` only).
- Absolute paths in Makefiles or Scripts (use relative paths).
- Waveform dumps (`dump.vcd`).
- Personal IDE settings (`vivado.jou` in root).
</anti_patterns>
