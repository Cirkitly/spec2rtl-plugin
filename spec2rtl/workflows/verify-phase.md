<purpose>
Verify phase goal achievement through goal-backward analysis. For Hardware Design, this includes checking RTL correctness, simulation results, and coverage metrics.

This workflow is executed by the s2r-uvm-engineer spawned from execute-phase.md.
</purpose>

<core_principle>
**Task completion ≠ Goal achievement**

A task "create ALU" can be marked complete when the file exists but has syntax errors. The task was done, but the goal "working ALU" was not achieved.

Goal-backward verification starts from the outcome and works backwards:
1. What must be TRUE (Simulation Pass, Coverage Met)?
2. What must EXIST (RTL, Testbench, Constraints)?
3. What must be WIRED (Instantiations, Clock/Reset trees)?

Then verify each level against the actual codebase.
</core_principle>

<required_reading>
@~/.claude/spec2rtl/references/eda-tools.md
@~/.claude/spec2rtl/templates/verification-report.md
</required_reading>

<process>

<step name="load_context" priority="first">
**Gather all verification context:**

```bash
# Phase directory (match both zero-padded and unpadded)
PADDED_PHASE=$(printf "%02d" ${PHASE_ARG} 2>/dev/null || echo "${PHASE_ARG}")
PHASE_DIR=$(ls -d .planning/phases/${PADDED_PHASE}-* .planning/phases/${PHASE_ARG}-* 2>/dev/null | head -1)

# Phase goal from ROADMAP
grep -A 5 "Phase ${PHASE_NUM}" .planning/ROADMAP.md

# Requirements mapped to this phase
grep -E "^| ${PHASE_NUM}" .planning/REQUIREMENTS.md 2>/dev/null

# All SUMMARY files (claims to verify)
ls "$PHASE_DIR"/*-SUMMARY.md 2>/dev/null

# All PLAN files (for must_haves in frontmatter)
ls "$PHASE_DIR"/*-PLAN.md 2>/dev/null
```

**Extract phase goal:** Parse ROADMAP.md for this phase's goal/description. This is the outcome to verify, not the tasks.

**Extract requirements:** If REQUIREMENTS.md exists, find requirements mapped to this phase.
</step>

<step name="establish_must_haves">
**Determine what must be verified.**

**Option A: Must-haves in PLAN frontmatter**

Check if any PLAN.md has `must_haves` in frontmatter:

```bash
grep -l "must_haves:" "$PHASE_DIR"/*-PLAN.md 2>/dev/null
```

If found, extract and use:
```yaml
must_haves:
  truths:
    - "ALU adds correctly"
    - "No X-propagation during reset"
  artifacts:
    - path: "rtl/alu.sv"
      provides: "ALU Logic"
  key_links:
    - from: "rtl/cpu_core.sv"
      to: "rtl/alu.sv"
      via: "instantiation"
```

**Option B: Derive from phase goal**

If no must_haves in frontmatter, derive using goal-backward process:

1. **State the goal:** Take phase goal from ROADMAP.md

2. **Derive truths:** Ask "What must be TRUE for this goal to be achieved?"
   - "Simulation passes with 0 errors"
   - "Coverage > 90%"
   - "Timing met (WNS > 0)"

3. **Derive artifacts:** For each truth, ask "What must EXIST?"
   - Map truths to concrete files (`rtl/alu.sv`, `tb/tb_alu.sv`)

4. **Derive key links:** For each artifact, ask "What must be CONNECTED?"
   - Identify critical wiring (Module A instantiates Module B)
   - Clock/Reset connectivity

5. **Document derived must-haves** before proceeding.
</step>

<step name="verify_truths">
**For each observable truth, determine if codebase enables it.**

**Verification status:**
- ✓ VERIFIED: Simulation passes / Code compiles / Assertions hold
- ✗ FAILED: Compilation error / Simulation fail / Missing file
- ? UNCERTAIN: Can't verify programmatically (needs human waveform inspection)

**For each truth:**
1. Identify supporting artifacts.
2. Check artifact status (see verify_artifacts step).
3. Check wiring status (see verify_wiring step).
4. **Check Simulation Logs:**
   - Look for `TEST PASSED` or `UVM_ERROR :    0`
   - Check exit codes of simulation scripts

**Example:**
Truth: "ALU adds correctly"
- Supporting artifacts: `rtl/alu.sv`, `tb/tb_alu.sv`
- If `tb_alu.sv` log says "TEST PASSED" → Truth VERIFIED
</step>

<step name="verify_artifacts">
**For each required artifact, verify three levels:**

### Level 1: Existence

```bash
check_exists() {
  local path="$1"
  [ -f "$path" ] && echo "EXISTS" || echo "MISSING"
}
```

### Level 2: Substantive

Check that the file has real implementation, not a stub.

```bash
check_length() {
  local path="$1"
  local min_lines="$2"
  local lines=$(wc -l < "$path" 2>/dev/null || echo 0)
  [ "$lines" -ge "$min_lines" ] && echo "SUBSTANTIVE" || echo "THIN"
}

check_stubs() {
  local path="$1"
  local stubs=$(grep -c -E "TODO|FIXME|placeholder" "$path" 2>/dev/null || echo 0)
  local empty=$(grep -c -E "module.*endmodule" "$path" 2>/dev/null || echo 0) # Check empty module

  [ "$stubs" -gt 0 ] && echo "STUB_PATTERNS" || echo "NO_STUBS"
}
```

### Level 3: Wired (Instantiated)

Check that the artifact is connected in the hierarchy.

```bash
check_instantiated() {
  local module_name="$1"
  local search_path="${2:-rtl/}"
  # Regex: module_name instance_name (
  local inst=$(grep -r "$module_name\s\+\w\+\s*(" "$search_path" 2>/dev/null | wc -l)
  [ "$inst" -gt 0 ] && echo "INSTANTIATED" || echo "ORPHANED"
}
```

**Final artifact status:**
- ✓ VERIFIED: Exists + Substantive + Instantiated
- ⚠️ ORPHANED: Exists + Substantive + Not Instantiated
- ✗ STUB: Thin or Stub patterns
- ✗ MISSING: Does not exist
</step>

<step name="verify_coverage">
**Check simulation coverage reports.**

1. **Locate coverage reports:**
   - Search for `.vdb`, `coverage/`, `urgReport/`.
   - Look for simulation logs mentioning "Coverage: X%".

2. **Verify coverage targets:**
   - [ ] Functional coverage meets target (>90% or as specified)
   - [ ] Code coverage (Line, Toggle, FSM) analyzed

3. **Status:**
   - ✓ VERIFIED: Coverage met
   - ✗ FAILED: Coverage low
   - ? UNCERTAIN: No report
</step>

<step name="verify_metrics">
**Check hardware metrics (if applicable).**

1. **Timing Verification:**
   - Check synthesis/PAR reports for "Timing met" or WNS > 0.

2. **Area/Resource Utilization:**
   - Check if LUTs/FFs/BRAMs are within budget.

3. **Status:**
   - ✓ VERIFIED: Met
   - ✗ FAILED: Violated
</step>

<step name="verify_wiring">
**Verify key links between artifacts.**

### Pattern: Instantiation
Check if Parent instantiates Child.

```bash
verify_instantiation() {
  local parent="$1"
  local child_module="$2"
  grep -E "$child_module\s+\w+\s*\(" "$parent" >/dev/null && echo "WIRED" || echo "NOT_WIRED"
}
```

### Pattern: Clock/Reset Propagation
Check if clk/rst are passed to instances.

```bash
verify_clk_rst() {
  local file="$1"
  # Check for .clk(clk) or .rst(rst) patterns
  grep -E "\.clk\s*\(|\.rst\s*\(" "$file" >/dev/null && echo "WIRED" || echo "POTENTIAL_MISSING_CLOCK"
}
```

### Pattern: Port Connection
Check for empty port connections `.*()`.

```bash
verify_ports() {
  local file="$1"
  grep -E "\.\w+\s*\(\s*\)" "$file" >/dev/null && echo "OPEN_PORTS" || echo "ALL_PORTS_CONNECTED"
}
```
</step>

<step name="scan_antipatterns">
**Scan for hardware anti-patterns.**

```bash
scan_antipatterns() {
  local files="$@"
  echo "## Anti-Patterns Found"

  for file in $files; do
    [ -f "$file" ] || continue

    # Latches (missing else/default)
    grep -n -E "always_comb|always @" "$file" | while read line; do
       # (Complex check, simplified for grep)
       # Use Verilator lint output if available instead
       echo "Check lint logs for latches in $file"
    done

    # Timescale missing
    grep -L "timescale" "$file" && echo "| $file | - | Missing timescale | ⚠️ Warning |"

    # Async reset sync check (simplified)
    # TODO/FIXME
    grep -n -E "TODO|FIXME" "$file" | while read line; do
       echo "| $file | $(echo $line | cut -d: -f1) | TODO found | ⚠️ Warning |"
    done
  done
}
```
</step>

<step name="identify_human_verification">
**Flag items that need human verification.**

- Waveform glitches (glitch-free reset?)
- CDC correctness (needs visual inspection of synchronizers?)
- Throughput/Latency performance (visualize bottleneck?)
</step>

<step name="determine_status">
**Calculate overall verification status.**

**Status: passed**
- All truths VERIFIED (Sim passes)
- All artifacts VERIFIED (Substantive + Wired)
- No blocker anti-patterns

**Status: gaps_found**
- Sim fails OR Artifact MISSING/STUB/ORPHANED
- Blocker anti-patterns (Latches)

**Status: human_needed**
- Automated checks pass, but human items remain
</step>

<step name="generate_fix_plans">
**If gaps_found, recommend fix plans.**
- "Implement missing ALU opcodes"
- "Fix latch in decoder"
- "Connect interrupt lines in Top"
</step>

<step name="create_report">
**Generate VERIFICATION.md using template.**
See `~/.claude/spec2rtl/templates/verification-report.md`.
</step>

<step name="return_to_orchestrator">
Return:
```markdown
## Verification Complete
**Status:** {passed | gaps_found | human_needed}
**Report:** .planning/phases/{phase_dir}/{phase}-VERIFICATION.md
```
</step>

</process>

<success_criteria>
- [ ] Must-haves established
- [ ] Truths verified (Simulation)
- [ ] Artifacts verified (RTL/TB existence & wiring)
- [ ] Coverage/Metrics checked
- [ ] Anti-patterns scanned
- [ ] VERIFICATION.md created
- [ ] Structured result returned
</success_criteria>
