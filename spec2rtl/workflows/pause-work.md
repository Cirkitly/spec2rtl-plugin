<purpose>
Capture current context and create a clean handoff point when pausing work.
</purpose>

<process>

<step name="capture_state">
**Identify current position:**

```bash
# Get current phase/plan from STATE.md or inference
grep "Current Position" .planning/STATE.md 2>/dev/null
```

**Find incomplete work:**
```bash
# Check for modified files
git status --porcelain
```

**Identify active agent:**
```bash
# Check if an agent was running (simulated check)
# In reality, the user is calling this, so no agent is running.
# But we might want to capture what was happening.
```
</step>

<step name="update_state">
**Update STATE.md with handoff info:**

```markdown
## Session Continuity

Last session: [current timestamp]
Stopped at: User paused work
Status: [WIP description]
```

**Create Handoff Note (Optional):**
If user provides a reason/note, append it to STATE.md.
</step>

<step name="create_handoff_commit">
**Commit WIP state:**

```bash
if [ -n "$(git status --porcelain)" ]; then
  git add .
  git commit -m "wip: pause session [timestamp]"
  echo "Created WIP commit."
else
  echo "Clean working tree. No WIP commit needed."
fi
```
</step>

<step name="report_handoff">
**Show handoff summary:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Spec2RTL ► WORK PAUSED
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✓ State saved to .planning/STATE.md
✓ WIP changes committed (if any)

**To resume:**
`/s2r:resume-work`

See you next time!
```
</step>

</process>
