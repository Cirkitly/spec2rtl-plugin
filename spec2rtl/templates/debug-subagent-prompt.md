# Debug Subagent Prompt Template

Template for spawning s2r-debugger agent. The agent contains all debugging expertise - this template provides problem context only.

---

## Template

```markdown
<objective>
Investigate issue: {issue_id}

**Summary:** {issue_summary}
</objective>

<symptoms>
expected: {expected}
actual: {actual}
errors: {errors}
reproduction: {reproduction}
timeline: {timeline}
</symptoms>

<mode>
symptoms_prefilled: {true_or_false}
goal: {find_root_cause_only | find_and_fix}
</mode>

<debug_file>
Create: .planning/debug/{slug}.md
</debug_file>

<instructions>
1. **Analyze Symptoms:** Look at Waveforms, Simulation Logs, or Timing Reports provided.
2. **Hypothesize:** What could cause this signal to be X or that Timing Path to fail?
3. **Verify:** Use `grep` on logs, or write a small reproduction TB if needed.
4. **Diagnose:** Is it RTL Logic, CDC, Timing, or Tool Setup?
</instructions>
```

---

## Placeholders

| Placeholder | Source | Example |
|-------------|--------|---------|
| `{issue_id}` | Orchestrator-assigned | `fifo-overflow` |
| `{issue_summary}` | User description | `Data loss at high throughput` |
| `{expected}` | From symptoms | `No data loss` |
| `{actual}` | From symptoms | `Last 2 words dropped` |
| `{errors}` | From symptoms | `Assertion failed: FIFO Overflow` |
| `{reproduction}` | From symptoms | `Run test_fifo_burst` |
| `{timeline}` | From symptoms | `After clock domain change` |
| `{goal}` | Orchestrator sets | `find_and_fix` |
| `{slug}` | Generated | `fifo-overflow` |

---

## Usage

**From /s2r:debug:**
```python
Task(
  prompt=filled_template,
  subagent_type="s2r-debugger",
  description="Debug {slug}"
)
```

**From diagnose-issues (Verification):**
```python
Task(prompt=template, subagent_type="s2r-debugger", description="Debug UAT-001")
```

---

## Continuation

For checkpoints, spawn fresh agent with:

```markdown
<objective>
Continue debugging {slug}. Evidence is in the debug file.
</objective>

<prior_state>
Debug file: @.planning/debug/{slug}.md
</prior_state>

<checkpoint_response>
**Type:** {checkpoint_type}
**Response:** {user_response}
</checkpoint_response>

<mode>
goal: {goal}
</mode>
```
