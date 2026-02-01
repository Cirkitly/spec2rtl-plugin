<purpose>
Execute a phase prompt (PLAN.md) and create the outcome summary (SUMMARY.md) using Verification-Driven Development (VDD).
</purpose>

<required_reading>
Read STATE.md before any operation to load project context.
Read config.json for planning behavior settings.

@~/.claude/spec2rtl/references/git-integration.md
@~/.claude/spec2rtl/references/eda-tools.md
</required_reading>

<process>

<step name="resolve_model_profile" priority="first">
Read model profile for agent spawning:

```bash
MODEL_PROFILE=$(cat .planning/config.json 2>/dev/null | grep -o '"model_profile"[[:space:]]*:[[:space:]]*"[^"]*"' | grep -o '"[^"]*"$' | tr -d '"' || echo "balanced")
```

Default to "balanced" if not set.

**Model lookup table:**

| Agent | quality | balanced | budget |
|-------|---------|----------|--------|
| s2r-executor | opus | sonnet | sonnet |

Store resolved model for use in Task calls below.
</step>

<step name="load_project_state">
Before any operation, read project state:

```bash
cat .planning/STATE.md 2>/dev/null
```

**If file exists:** Parse and internalize:
- Current position (phase, plan, status)
- Accumulated decisions (constraints on this execution)
- Blockers/concerns (PPA constraints, Timing issues)

**If file missing but .planning/ exists:**
Reconstruct from existing artifacts or continue without project state.

**If .planning/ doesn't exist:** Error - project not initialized.

**Load planning config:**
```bash
COMMIT_PLANNING_DOCS=$(cat .planning/config.json 2>/dev/null | grep -o '"commit_docs"[[:space:]]*:[[:space:]]*[^,}]*' | grep -o 'true\|false' || echo "true")
git check-ignore -q .planning 2>/dev/null && COMMIT_PLANNING_DOCS=false
```
</step>

<step name="identify_plan">
Find the next plan to execute:
- Check roadmap for "In progress" phase
- Find plans in that phase directory
- Identify first plan without corresponding SUMMARY

```bash
cat .planning/ROADMAP.md
ls .planning/phases/XX-name/*-PLAN.md 2>/dev/null | sort
ls .planning/phases/XX-name/*-SUMMARY.md 2>/dev/null | sort
```

**Logic:**
- If `01-01-PLAN.md` exists but `01-01-SUMMARY.md` doesn't → execute 01-01
- Pattern: Find first PLAN file without matching SUMMARY file

<if mode="interactive">
Present found plan and wait for confirmation.
</if>
</step>

<step name="record_start_time">
```bash
PLAN_START_TIME=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
PLAN_START_EPOCH=$(date +%s)
```
</step>

<step name="parse_segments">
**Intelligent segmentation: Parse plan into execution segments.**

**1. Check for checkpoints:**
```bash
grep -n "type=\"checkpoint" .planning/phases/XX-name/{phase}-{plan}-PLAN.md
```

**2. Analyze execution strategy:**
- **Fully autonomous plan:** No checkpoints. Spawn single subagent.
- **Segmented:** Checkpoints exist. Route segments to subagents or main context.

**Routing Rules:**
- Segment with no prior checkpoint → SUBAGENT
- Segment after `checkpoint:human-verify` → SUBAGENT
- Segment after `checkpoint:decision` → MAIN CONTEXT

**3. Execution:**
See `segment_execution` step for detailed loop.
</step>

<step name="init_agent_tracking">
Initialize `.planning/agent-history.json` and clear `.planning/current-agent-id.txt`.
Check for interrupted agents to resume.
</step>

<step name="segment_execution">
**Detailed segment execution loop for segmented plans.**

For each segment:
1. Determine routing (Subagent vs Main).
2. If Subagent:
   - Spawn Task tool with `s2r-executor`.
   - Track agent ID.
   - Wait for completion.
3. If Main:
   - Execute tasks in main context.
4. Handle Checkpoints (present to user, wait for response).

After all segments:
- Aggregate results.
- Create SUMMARY.md.
- Commit.
</step>

<step name="load_prompt">
Read the plan prompt:
```bash
cat .planning/phases/XX-name/{phase}-{plan}-PLAN.md
```
This IS the execution instructions. Follow it exactly.
</step>

<step name="previous_phase_check">
Check if previous phase had issues (timing violations, unclosed gaps).
If yes, ask user how to proceed.
</step>

<step name="execute">
Execute each task in the prompt. **Deviations are normal** - handle them automatically.

1. Read @context files.

2. For each task:

   **If `type="auto"`:**

   **Before executing:** Check if task has `type="vdd"` or `vdd="true"` attribute:
   - **If yes:** Follow VDD execution flow (see `<vdd_execution>`) - ASSERT → IMPLEMENT → VERIFY cycle.
   - **If no:** Standard implementation.

   - Work toward task completion.
   - **If EDA Tool Error:** Handle as Environment Gate (e.g., missing license).
   - **If Deviations:** Apply rules (Fix bugs, Add missing reset logic).
   - Run verification (Simulation/Lint).
   - Confirm done criteria met.
   - **Commit the task** (see `<task_commit>`).
   - Track task completion.

   **If `type="checkpoint:*"`:**
   - STOP immediately.
   - Execute checkpoint_protocol.
   - Wait for user response.

3. Run overall verification checks.
4. Confirm success criteria.
5. Document deviations in Summary.
</step>

<environment_gates>
## Handling Environment/Tool Errors

**Indicators:**
- "License check failed"
- "Command not found: vivado"
- "Variable XILINX_VIVADO not set"

**Protocol:**
1. **Recognize gate** - Not a bug, environment issue.
2. **STOP task**.
3. **Create checkpoint:human-action**.
4. **Provide setup steps** (`source settings.sh`, `export LICENSE=...`).
5. **Wait for user**.
6. **Verify and Retry**.
</environment_gates>

<deviation_rules>
**RULE 1: Auto-fix bugs** (Logic errors, connectivity mismatches, syntax errors).
**RULE 2: Auto-add missing critical functionality** (Reset logic, clock synchronizers, standard compliance).
**RULE 3: Auto-fix blocking issues** (Missing includes, undefined macros).
**RULE 4: Ask about architectural changes** (Changing bus width, adding clock domains).
</deviation_rules>

<vdd_execution>
## VDD Plan Execution (Verification Driven Development)

When executing a task with `type="vdd"`, follow the ASSERT-IMPLEMENT-VERIFY cycle.

**1. Check verification infrastructure:**
- Detect simulator (Verilator, Icarus, Vivado).
- Ensure `Makefile` or `run.sh` exists.

**2. ASSERT - Write failing checks:**
- Create Testbench (`tb_module.sv`) or Bind file (`module_sva.sv`).
- Write **SVA (SystemVerilog Assertions)** or self-checking scoreboards.
- Run simulation - **MUST FAIL** (or compile error if RTL missing).
- Commit: `test({phase}-{plan}): add assertions/tb for [feature]`

**3. IMPLEMENT - Implement to pass:**
- Write RTL (`module.sv`) to satisfy assertions.
- Run simulation - **MUST PASS**.
- Commit: `feat({phase}-{plan}): implement [feature]`

**4. VERIFY/REFACTOR:**
- Optimize Area/Timing.
- Fix Lint warnings.
- Run simulation - **MUST STILL PASS**.
- Commit: `refactor({phase}-{plan}): optimize [feature]`

**Commit pattern:** 2-3 atomic commits per VDD task.
</vdd_execution>

<task_commit>
## Task Commit Protocol

1. **Identify modified files:** `git status --short`
2. **Stage files:** `git add ...` (Targeted)
3. **Commit:**
   `feat` (RTL), `test` (TB), `fix` (Bug), `refactor` (Opt), `chore` (Build).
   Format: `{type}({phase}-{plan}): {description}`
4. **Record hash** for Summary.
</task_commit>

<checkpoint_protocol>
**checkpoint:human-verify**: "I ran simulation. Check waveform at `dump.vcd`."
**checkpoint:decision**: "Choose Arbiter scheme: Round-Robin vs Fixed Priority."
**checkpoint:human-action**: "Please source Vivado license file."
</checkpoint_protocol>

<step name="generate_user_setup">
**Generate USER-SETUP.md if plan has user_setup.**

Content:
- **Environment Variables**: `LM_LICENSE_FILE`, `XILINX_PATH`.
- **Tool Setup**: Installation paths, vendor libraries.
- **Verification**: `vlog --version` or equivalent.
</step>

<step name="create_summary">
Create `{phase}-{plan}-SUMMARY.md`.

**Frontmatter:**
- phase, plan, subsystem
- requires, provides, affects
- tech-stack (IPs used)
- metrics (Area, FMax, Duration)

**Content:**
- Accomplishments (RTL modules, Testbenches)
- Verification Results (Passing tests, Coverage %)
- Deviations
</step>

<step name="update_state_and_roadmap">
Update STATE.md (Progress, Decisions).
Update ROADMAP.md (Status).
</step>

<step name="git_commit_metadata">
Commit SUMMARY, STATE, ROADMAP.
Message: `docs({phase}-{plan}): complete [plan-name] plan`
</step>

<step name="offer_next">
**Verify completion:**
- Count plans vs summaries.
- If more plans: Execute next.
- If phase complete: Check milestone.
- Present next logical command (`/s2r:execute-phase` or `/s2r:complete-milestone`).
</step>

</process>

<success_criteria>
- [ ] All tasks executed (using VDD where specified)
- [ ] Atomic commits for Tests/RTL
- [ ] Simulation passed for all tasks
- [ ] SUMMARY.md created with metrics
- [ ] STATE.md/ROADMAP.md updated
- [ ] Metadata committed
</success_criteria>
