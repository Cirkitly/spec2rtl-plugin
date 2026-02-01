<purpose>
Orchestrate parallel debug agents to investigate Verification gaps and find root causes in RTL/TB.

After Verification finds gaps (Sim failures, Coverage holes), spawn one debug agent per gap. Each agent investigates autonomously using simulation logs and waveforms. Collect root causes, update VERIFICATION.md gaps with diagnosis, then hand off to plan-phase --gaps.
</purpose>

<paths>
DEBUG_DIR=.planning/debug

Debug files use the `.planning/debug/` path.
</paths>

<core_principle>
**Diagnose before planning fixes.**

Verification tells us WHAT is broken (symptoms: "Test failed at time 450ns"). Debug agents find WHY (root cause: "Reset polarity mismatch in sub-module").

Without diagnosis: "ALU Output Wrong" → guess at fix → "rewrite ALU" (wasteful)
With diagnosis: "ALU Output Wrong" → "Opcode 4 decoded incorrectly" → "Fix Case Statement" (precise)
</core_principle>

<process>

<step name="parse_gaps">
**Extract gaps from VERIFICATION.md:**

Read the "Gaps" section (YAML format):
```yaml
- truth: "ALU performs addition correctly"
  status: failed
  reason: "Simulation mismatch: Expected 0x8, Got 0x0"
  severity: major
  artifacts: ["rtl/alu.sv"]
  missing: []
```

Build gap list for parallel execution.
</step>

<step name="report_plan">
**Report diagnosis plan to user:**

```
## Diagnosing {N} Gaps

Spawning parallel debug agents to investigate root causes:

| Gap (Truth) | Severity |
|-------------|----------|
| ALU performs addition correctly | major |
| FIFO full flag asserts early | minor |
| Reset clears internal state | blocker |

Each agent will:
1. Analyze simulation logs (transcript)
2. Inspect waveforms (VCD/FST) if available
3. Trace signals backwards from failure point
4. Return root cause
```
</step>

<step name="spawn_agents">
**Spawn debug agents in parallel:**

For each gap, spawn a task:

```
Task(
  prompt="Diagnose verification gap: {truth}

  Reason: {reason}
  Artifacts: {artifacts}

  1. Locate the failure in simulation logs (look for UVM_ERROR or Error:).
  2. If a waveform dump exists (dump.vcd), inspect signal states at the failure time.
  3. Read the relevant RTL and Testbench files.
  4. Trace the data path backwards from the failing output.
  5. Check for common RTL bugs:
     - Reset polarity mismatch
     - Width mismatches (truncation)
     - Sensitivity list omissions
     - Blocking vs Non-blocking assignment mix
     - CDC issues (X propagation)
  6. Determine Root Cause.
  7. Suggest specific RTL fix.",
  subagent_type="s2r-debugger",
  description="Debug: {truth_short}"
)
```

**All agents spawn in single message** (parallel execution).
</step>

<step name="collect_results">
**Collect root causes from agents:**

Each agent returns with:
```
## ROOT CAUSE FOUND

**Debug Session:** ${DEBUG_DIR}/{slug}.md

**Root Cause:** {specific cause with evidence}

**Evidence Summary:**
- {log snippet}
- {waveform observation}

**Files Involved:**
- {file1}: {what's wrong}

**Suggested Fix:** {specific RTL change}
```

Parse each return to extract root_cause and suggested_fix.
</step>

<step name="update_verification">
**Update VERIFICATION.md gaps with diagnosis:**

For each gap in the Gaps section, add artifacts and missing fields:

```yaml
- truth: "ALU performs addition correctly"
  status: failed
  reason: "Simulation mismatch: Expected 0x8, Got 0x0"
  root_cause: "Carry-in signal not connected in adder instance"
  missing:
    - "Connect cin port in alu.sv line 45"
  debug_session: .planning/debug/alu-addition-fail.md
```

**Check planning config for commit preference.**
If enabled, commit the updated VERIFICATION.md.
</step>

<step name="report_results">
**Report diagnosis results and hand off:**

Display:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Spec2RTL ► DIAGNOSIS COMPLETE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

| Gap | Root Cause | Files |
|-----|------------|-------|
| ALU addition | Carry-in disconnected | alu.sv |
| FIFO full | Off-by-one in pointer logic | fifo.sv |

Proceeding to plan fixes...
```

Return to orchestrator for automatic planning.
</step>

</process>

<failure_handling>
**Agent fails to find root cause:**
- Mark gap as "needs manual waveform inspection"
- Report incomplete diagnosis

**Tools missing:**
- If simulator not available, perform static code analysis only.
</failure_handling>

<success_criteria>
- [ ] Gaps parsed from VERIFICATION.md
- [ ] Debug agents spawned in parallel
- [ ] Root causes collected
- [ ] VERIFICATION.md updated
- [ ] Hand off to plan-phase --gaps
</success_criteria>
