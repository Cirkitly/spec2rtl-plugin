<purpose>
Generate detailed execution plans (PLAN.md) for a roadmap phase. Uses a specialized Hardware Planner agent to decompose goals into RTL/Verification tasks and wave-based schedules.
</purpose>

<core_principle>
Planning is architectural work. It requires understanding the "What" (Goal) and defining the "How" (Micro-architecture, Interfaces, Verification Strategy). It does NOT implement code.
</core_principle>

<required_reading>
Read STATE.md and ROADMAP.md before planning.
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
| s2r-planner | opus | sonnet | sonnet |
| general-purpose | — | — | — |

Store resolved models.
</step>

<step name="validate_phase">
Ensure phase exists in roadmap and file system.

```bash
# Normalize input
PADDED_PHASE=$(printf "%02d" ${PHASE_ARG} 2>/dev/null || echo "${PHASE_ARG}")
PHASE_DIR=$(ls -d .planning/phases/${PADDED_PHASE}-* .planning/phases/${PHASE_ARG}-* 2>/dev/null | head -1)

if [ -z "$PHASE_DIR" ]; then
  # Try to find in ROADMAP.md to create directory if missing (but planned)
  PHASE_NAME=$(grep -i "Phase ${PHASE_ARG}:" .planning/ROADMAP.md | head -1 | sed 's/.*: //' | tr -d '[]')

  if [ -n "$PHASE_NAME" ]; then
    # Create directory structure
    SLUG=$(echo "$PHASE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
    PHASE_DIR=".planning/phases/${PADDED_PHASE}-${SLUG}"
    mkdir -p "$PHASE_DIR"
    echo "Created phase directory: $PHASE_DIR"
  else
    echo "ERROR: Phase ${PHASE_ARG} not found in ROADMAP.md or .planning/phases/"
    exit 1
  fi
fi
```
</step>

<step name="gather_context">
Ensure necessary context exists before planning.

**Check 1: Context Definition (Micro-arch)**
If `{phase}-CONTEXT.md` is missing, we need to define the micro-architecture.
Use `discuss-phase` skill (or prompt user) to generate it.

```bash
CONTEXT_FILE="$PHASE_DIR/${PADDED_PHASE}-CONTEXT.md"
if [ ! -f "$CONTEXT_FILE" ]; then
  echo "MISSING: $CONTEXT_FILE"
  echo "Running context gathering..."
  # In a real shell execution, this would trigger /s2r:discuss-phase
  # For now, we will create a skeleton or prompt the user
  # For the agent workflow, we'll mark this as a task for the planner to request
fi
```

**Check 2: Research/Discovery**
If the phase involves complex IP selection or unknown protocols, `RESEARCH.md` or `DISCOVERY.md` should exist.
</step>

<step name="surface_assumptions">
Run assumptions check to prevent "hallucinated specs".

```bash
# This would invoke the list-phase-assumptions workflow
# Output is captured for the planner context
```
</step>

<step name="spawn_planner">
Spawn the `s2r-planner` agent to generate plans.

**Inputs:**
- Phase Number/Name
- Project State (PPA goals, Clock speeds)
- Roadmap (Dependencies)
- Context (Micro-arch)

**Template:** `templates/planner-subagent-prompt.md`

```python
# Construct prompt using the template
# Read files to inject content
```

Task(
    subagent_type="s2r-planner",
    prompt="...",
    description="Plan Phase ${PHASE_ARG}"
)
</step>

<step name="post_plan_review">
Display generated plans and validation status.

```bash
ls -1 "$PHASE_DIR"/*-PLAN.md
```

**Route to Execute:**
"Planning complete. {N} plans created.
Next: `/s2r:execute-phase ${PHASE_ARG}`"
</step>

</process>

<usage_guide>
`gsd:plan-phase {N}`: Plan phase N.
`gsd:plan-phase {N} --gaps`: Plan gap closure for phase N (after verification failure).
</usage_guide>
