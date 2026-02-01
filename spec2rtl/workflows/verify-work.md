     1→<purpose>
     2→Validate built hardware through conversational testing with persistent state. Creates UAT.md (Manual Verification Log) that tracks inspection progress, survives /clear, and feeds gaps into /s2r:plan-phase --gaps.
     3→
     4→User checks Waveforms/FPGA, Claude records. One item at a time. Plain text responses.
     5→</purpose>
     6→
     7→<philosophy>
     8→**Show expected behavior, ask if reality matches.**
     9→
    10→Claude presents what SHOULD be observed in the Waveform Viewer or Lab Board. User confirms or describes the discrepancy.
    11→- "yes" / "y" / "looks good" / "clean" → pass
    12→- Anything else → logged as issue, severity inferred
    13→
    14→No Pass/Fail buttons. No severity questions. Just: "Here's what the signal should do. Does it?"
    15→</philosophy>
    16→
    17→<template>
    18→@~/.claude/spec2rtl/templates/UAT.md
    19→</template>
    20→
    21→<process>
    22→
    23→<step name="resolve_model_profile" priority="first">
    24→Read model profile for agent spawning:
    25→
    26→```bash
    27→MODEL_PROFILE=$(cat .planning/config.json 2>/dev/null | grep -o '"model_profile"[[:space:]]*:[[:space:]]*"[^"]*"' | grep -o '"[^"]*"$' | tr -d '"' || echo "balanced")
    28→```
    29→
    30→Default to "balanced" if not set.
    31→
    32→**Model lookup table:**
    33→
    34→| Agent | quality | balanced | budget |
    35→|-------|---------|----------|--------|
    36→| s2r-planner | opus | opus | sonnet |
    37→| s2r-plan-checker | sonnet | sonnet | haiku |
    38→
    39→Store resolved models for use in Task calls below.
    40→</step>
    41→
    42→<step name="check_active_session">
    43→**First: Check for active Manual Verification sessions**
    44→
    45→```bash
    46→find .planning/phases -name "*-UAT.md" -type f 2>/dev/null | head -5
    47→```
    48→
    49→**If active sessions exist AND no $ARGUMENTS provided:**
    50→
    51→Read each file's frontmatter (status, phase) and Current Item section.
    52→
    53→Display inline:
    54→
    55→```
    56→## Active Verification Sessions
    57→
    58→| # | Phase | Status | Current Item | Progress |
    59→|---|-------|--------|--------------|----------|
    60→| 1 | 04-uart | testing | 3. Check TX Waveform | 2/6 |
    61→| 2 | 05-ddr | testing | 1. Calibration LED | 0/4 |
    62→
    63→Reply with a number to resume, or provide a phase number to start new.
    64→```
    65→
    66→Wait for user response.
    67→
    68→- If user replies with number (1, 2) → Load that file, go to `resume_from_file`
    69→- If user replies with phase number → Treat as new session, go to `create_uat_file`
    70→
    71→**If active sessions exist AND $ARGUMENTS provided:**
    72→
    73→Check if session exists for that phase. If yes, offer to resume or restart.
    74→If no, continue to `create_uat_file`.
    75→
    76→**If no active sessions AND no $ARGUMENTS:**
    77→
    78→```
    79→No active sessions.
    80→
    81→Provide a phase number to start verification (e.g., /s2r:verify-work 4)
    82→```
    83→
    84→**If no active sessions AND $ARGUMENTS provided:**
    85→
    86→Continue to `create_uat_file`.
    87→</step>
    88→
    89→<step name="find_summaries">
    90→**Find what to verify:**
    91→
    92→Parse $ARGUMENTS as phase number (e.g., "4") or plan number (e.g., "04-02").
    93→
    94→```bash
    95→# Find phase directory (match both zero-padded and unpadded)
    96→PADDED_PHASE=$(printf "%02d" ${PHASE_ARG} 2>/dev/null || echo "${PHASE_ARG}")
    97→PHASE_DIR=$(ls -d .planning/phases/${PADDED_PHASE}-* .planning/phases/${PHASE_ARG}-* 2>/dev/null | head -1)
    98→
    99→# Find SUMMARY files
   100→ls "$PHASE_DIR"/*-SUMMARY.md 2>/dev/null
   101→```
   102→
   103→Read each SUMMARY.md to extract verifiable deliverables.
   104→</step>
   105→
   106→<step name="extract_tests">
   107→**Extract inspection items from SUMMARY.md:**
   108→
   109→Parse for:
   110→1. **RTL Modules** - Need waveform inspection
   111→2. **External Interfaces** - Need IO verification
   112→3. **Features** - Need functional observation
   113→
   114→Focus on OBSERVABLE hardware behavior (Waveforms or LEDs).
   115→
   116→For each deliverable, create an item:
   117→- name: Brief item name
   118→- expected: What the user should see in VCD/Board (specific, observable)
   119→
   120→Examples:
   121→- Deliverable: "UART Transmit Logic"
   122→  → Item: "Check UART TX Waveform"
   123→  → Expected: "Assert valid_i. Expect tx_o to toggle at baud rate. Ready_o should drop while busy."
   124→
   125→Skip internal implementation details (e.g., "Optimized state encoding").
   126→</step>
   127→
   128→<step name="create_uat_file">
   129→**Create Manual Verification Log with all items:**
   130→
   131→```bash
   132→mkdir -p "$PHASE_DIR"
   133→```
   134→
   135→Build item list from extracted deliverables.
   136→
   137→Create file:
   138→
   139→```markdown
   140→---
   141→status: testing
   142→phase: XX-name
   143→source: [list of SUMMARY.md files]
   144→started: [ISO timestamp]
   145→updated: [ISO timestamp]
   146→---
   147→
   148→## Current Item
   149→<!-- OVERWRITE each item - shows where we are -->
   150→
   151→number: 1
   152→name: [first item name]
   153→expected: |
   154→  [what user should observe in waveform/board]
   155→awaiting: user response
   156→
   157→## Inspection Items
   158→
   159→### 1. [Item Name]
   160→expected: [observable behavior]
   161→result: [pending]
   162→
   163→### 2. [Item Name]
   164→expected: [observable behavior]
   165→result: [pending]
   166→
   167→...
   168→
   169→## Summary
   170→
   171→total: [N]
   172→passed: 0
   173→issues: 0
   174→pending: [N]
   175→skipped: 0
   176→
   177→## Gaps
   178→
   179→[none yet]
   180→```
   181→
   182→Write to `.planning/phases/XX-name/{phase}-UAT.md`
   183→
   184→Proceed to `present_test`.
   185→</step>
   186→
   187→<step name="present_test">
   188→**Present current item to user:**
   189→
   190→Read Current Item section from file.
   191→
   192→Display using checkpoint box format:
   193→
   194→```
   195→╔══════════════════════════════════════════════════════════════╗
   196→║  CHECKPOINT: Manual Verification Required                    ║
   197→╚══════════════════════════════════════════════════════════════╝
   198→
   199→**Item {number}: {name}**
   200→
   201→{expected}
   202→
   203→──────────────────────────────────────────────────────────────
   204→→ Type "pass" or describe what you see (Waveform/Lab)
   205→──────────────────────────────────────────────────────────────
   206→```
   207→
   208→Wait for user response (plain text, no AskUserQuestion).
   209→</step>
   210→
   211→<step name="process_response">
   212→**Process user response and update file:**
   213→
   214→**If response indicates pass:**
   215→- Empty response, "yes", "y", "ok", "pass", "clean", "good", "✓"
   216→
   217→Update Tests section:
   218→```
   219→### {N}. {name}
   220→expected: {expected}
   221→result: pass
   222→```
   223→
   224→**If response indicates skip:**
   225→- "skip", "no board", "can't check", "n/a"
   226→
   227→Update Tests section:
   228→```
   229→### {N}. {name}
   230→expected: {expected}
   231→result: skipped
   232→reason: [user's reason if provided]
   233→```
   234→
   235→**If response is anything else:**
   236→- Treat as issue description
   237→
   238→Infer severity from description:
   239→- Contains: crash, hang, deadlock, X propagation, unknown, reset fail → blocker
   240→- Contains: mismatch, wrong value, timing violation, protocol error → major
   241→- Contains: glitch, jitter, noise, warning → minor
   242→- Default if unclear: major
   243→
   244→Update Tests section:
   245→```
   246→### {N}. {name}
   247→expected: {expected}
   248→result: issue
   249→reported: "{verbatim user response}"
   250→severity: {inferred}
   251→```
   252→
   253→Append to Gaps section (structured YAML for plan-phase --gaps):
   254→```yaml
   255→- truth: "{expected behavior from test}"
   256→  status: failed
   257→  reason: "User observed: {verbatim user response}"
   258→  severity: {inferred}
   259→  test: {N}
   260→  artifacts: []  # Filled by diagnosis
   261→  missing: []    # Filled by diagnosis
   262→```
   263→
   264→**After any response:**
   265→
   266→Update Summary counts.
   267→Update frontmatter.updated timestamp.
   268→
   269→If more items remain → Update Current Item, go to `present_test`
   270→If no more items → Go to `complete_session`
   271→</step>
   272→
   273→<step name="resume_from_file">
   274→**Resume verification from file:**
   275→
   276→Read the full UAT file.
   277→
   278→Find first item with `result: [pending]`.
   279→
   280→Announce:
   281→```
   282→Resuming: Phase {phase} Verification
   283→Progress: {passed + issues + skipped}/{total}
   284→Issues found so far: {issues count}
   285→
   286→Continuing from Item {N}...
   287→```
   288→
   289→Update Current Item section with the pending item.
   290→Proceed to `present_test`.
   291→</step>
   292→
   293→<step name="complete_session">
   294→**Complete testing and commit:**
   295→
   296→Update frontmatter:
   297→- status: complete
   298→- updated: [now]
   299→
   300→Clear Current Item section:
   301→```
   302→## Current Item
   303→
   304→[verification complete]
   305→```
   306→
   307→**Check planning config:**
   308→
   309→```bash
   310→COMMIT_PLANNING_DOCS=$(cat .planning/config.json 2>/dev/null | grep -o '"commit_docs"[[:space:]]*:[[:space:]]*[^,}]*' | grep -o 'true\|false' || echo "true")
   311→git check-ignore -q .planning 2>/dev/null && COMMIT_PLANNING_DOCS=false
   312→```
   313→
   314→**If `COMMIT_PLANNING_DOCS=false`:** Skip git operations
   315→
   316→**If `COMMIT_PLANNING_DOCS=true` (default):**
   317→
   318→Commit the UAT file:
   319→```bash
   320→git add ".planning/phases/XX-name/{phase}-UAT.md"
   321→git commit -m "test({phase}): complete manual verification - {passed} passed, {issues} issues"
   322→```
   323→
   324→Present summary:
   325→```
   326→## Verification Complete: Phase {phase}
   327→
   328→| Result | Count |
   329→|--------|-------|
   330→| Passed | {N}   |
   331→| Issues | {N}   |
   332→| Skipped| {N}   |
   333→
   334→[If issues > 0:]
   335→### Issues Found
   336→
   337→[List from Issues section]
   338→```
   339→
   340→**If issues > 0:** Proceed to `diagnose_issues`
   341→
   342→**If issues == 0:**
   343→```
   344→All checks passed. Ready to continue.
   345→
   346→- `/s2r:plan-phase {next}` — Plan next phase
   347→- `/s2r:execute-phase {next}` — Execute next phase
   348→```
   349→</step>
   350→
   351→<step name="diagnose_issues">
   352→**Diagnose root causes before planning fixes:**
   353→
   354→```
   355→---
   356→
   357→{N} issues found. Diagnosing root causes...
   358→
   359→Spawning parallel debug agents to investigate each issue.
   360→```
   361→
   362→- Load diagnose-issues workflow
   363→- Follow @~/.claude/spec2rtl/workflows/diagnose-issues.md
   364→- Spawn parallel debug agents for each issue
   365→- Collect root causes
   366→- Update UAT.md with root causes
   367→- Proceed to `plan_gap_closure`
   368→
   369→Diagnosis runs automatically - no user prompt. Parallel agents investigate simultaneously, so overhead is minimal and fixes are more accurate.
   370→</step>
   371→
   372→<step name="plan_gap_closure">
   373→**Auto-plan fixes from diagnosed gaps:**
   374→
   375→Display:
   376→```
   377→━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   378→ Spec2RTL ► PLANNING FIXES
   379→━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   380→
   381→◆ Spawning planner for gap closure...
   382→```
   383→
   384→Spawn s2r-planner in --gaps mode:
   385→
   386→```
   387→Task(
   388→  prompt="""
   389→<planning_context>
   390→
   391→**Phase:** {phase_number}
   392→**Mode:** gap_closure
   393→
   394→**Verification Log with diagnoses:**
   395→@.planning/phases/{phase_dir}/{phase}-UAT.md
   396→
   397→**Project State:**
   398→@.planning/STATE.md
   399→
   400→**Roadmap:**
   401→@.planning/ROADMAP.md
   402→
   403→</planning_context>
   404→
   405→<downstream_consumer>
   406→Output consumed by /s2r:execute-phase
   407→Plans must be executable prompts.
   408→</downstream_consumer>
   409→""",
   410→  subagent_type="s2r-planner",
   411→  model="{planner_model}",
   412→  description="Plan gap fixes for Phase {phase}"
   413→)
   414→```
   415→
   416→On return:
   417→- **PLANNING COMPLETE:** Proceed to `verify_gap_plans`
   418→- **PLANNING INCONCLUSIVE:** Report and offer manual intervention
   419→</step>
   420→
   421→<step name="verify_gap_plans">
   422→**Verify fix plans with checker:**
   423→
   424→Display:
   425→```
   426→━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   427→ Spec2RTL ► VERIFYING FIX PLANS
   428→━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   429→
   430→◆ Spawning plan checker...
   431→```
   432→
   433→Initialize: `iteration_count = 1`
   434→
   435→Spawn s2r-plan-checker:
   436→
   437→```
   438→Task(
   439→  prompt="""
   440→<verification_context>
   441→
   442→**Phase:** {phase_number}
   443→**Phase Goal:** Close diagnosed gaps from verification
   444→
   445→**Plans to verify:**
   446→@.planning/phases/{phase_dir}/*-PLAN.md
   447→
   448→</verification_context>
   449→
   450→<expected_output>
   451→Return one of:
   452→- ## VERIFICATION PASSED — all checks pass
   453→- ## ISSUES FOUND — structured issue list
   454→</expected_output>
   455→""",
   456→  subagent_type="s2r-plan-checker",
   457→  model="{checker_model}",
   458→  description="Verify Phase {phase} fix plans"
   459→)
   460→```
   461→
   462→On return:
   463→- **VERIFICATION PASSED:** Proceed to `present_ready`
   464→- **ISSUES FOUND:** Proceed to `revision_loop`
   465→</step>
   466→
   467→<step name="revision_loop">
   468→**Iterate planner ↔ checker until plans pass (max 3):**
   469→
   470→**If iteration_count < 3:**
   471→
   472→Display: `Sending back to planner for revision... (iteration {N}/3)`
   473→
   474→Spawn s2r-planner with revision context:
   475→
   476→```
   477→Task(
   478→  prompt="""
   479→<revision_context>
   480→
   481→**Phase:** {phase_number}
   482→**Mode:** revision
   483→
   484→**Existing plans:**
   485→@.planning/phases/{phase_dir}/*-PLAN.md
   486→
   487→**Checker issues:**
   488→{structured_issues_from_checker}
   489→
   490→</revision_context>
   491→
   492→<instructions>
   493→Read existing PLAN.md files. Make targeted updates to address checker issues.
   494→Do NOT replan from scratch unless issues are fundamental.
   495→</instructions>
   496→""",
   497→  subagent_type="s2r-planner",
   498→  model="{planner_model}",
   499→  description="Revise Phase {phase} plans"
   500→)
   501→```
   502→
   503→After planner returns → spawn checker again (verify_gap_plans logic)
   504→Increment iteration_count
   505→
   506→**If iteration_count >= 3:**
   507→
   508→Display: `Max iterations reached. {N} issues remain.`
   509→
   510→Offer options:
   511→1. Force proceed (execute despite issues)
   512→2. Provide guidance (user gives direction, retry)
   513→3. Abandon (exit, user runs /s2r:plan-phase manually)
   514→
   515→Wait for user response.
   516→</step>
   517→
   518→<step name="present_ready">
   519→**Present completion and next steps:**
   520→
   521→```
   522→━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   523→ Spec2RTL ► FIXES READY ✓
   524→━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
   525→
   526→**Phase {X}: {Name}** — {N} gap(s) diagnosed, {M} fix plan(s) created
   527→
   528→| Gap | Root Cause | Fix Plan |
   529→|-----|------------|----------|
   530→| {truth 1} | {root_cause} | {phase}-04 |
   531→| {truth 2} | {root_cause} | {phase}-04 |
   532→
   533→Plans verified and ready for execution.
   534→
   535→───────────────────────────────────────────────────────────────
   536→
   537→## ▶ Next Up
   538→
   539→**Execute fixes** — run fix plans
   540→
   541→`/clear` then `/s2r:execute-phase {phase} --gaps-only`
   542→
   543→───────────────────────────────────────────────────────────────
   544→```
   545→</step>
   546→
   547→</process>
   548→
   549→<update_rules>
   550→**Batched writes for efficiency:**
   551→
   552→Keep results in memory. Write to file only when:
   553→1. **Issue found** — Preserve the problem immediately
   554→2. **Session complete** — Final write before commit
   555→3. **Checkpoint** — Every 5 passed items (safety net)
   556→
   557→| Section | Rule | When Written |
   558→|---------|------|--------------|
   559→| Frontmatter.status | OVERWRITE | Start, complete |
   560→| Frontmatter.updated | OVERWRITE | On any file write |
   561→| Current Item | OVERWRITE | On any file write |
   562→| Tests.{N}.result | OVERWRITE | On any file write |
   563→| Summary | OVERWRITE | On any file write |
   564→| Gaps | APPEND | When issue found |
   565→
   566→On context reset: File shows last checkpoint. Resume from there.
   567→</update_rules>
   568→
   569→<severity_inference>
   570→**Infer severity from user's natural language:**
   571→
   572→| User says | Infer |
   573→|-----------|-------|
   574→| "crash", "hang", "deadlock", "X propagation" | blocker |
   575→| "mismatch", "wrong value", "timing violation" | major |
   576→| "glitch", "warning", "noise" | minor |
   577→
   578→Default to **major** if unclear. User can correct if needed.
   579→
   580→**Never ask "how severe is this?"** - just infer and move on.
   581→</severity_inference>
   582→
   583→<success_criteria>
   584→- [ ] UAT file created with all items from SUMMARY.md
   585→- [ ] Items presented one at a time with expected waveform/lab behavior
   586→- [ ] User responses processed as pass/issue/skip
   587→- [ ] Severity inferred from description (never asked)
   588→- [ ] Batched writes: on issue, every 5 passes, or completion
   589→- [ ] Committed on completion
   590→- [ ] If issues: parallel debug agents diagnose root causes
   591→- [ ] If issues: s2r-planner creates fix plans (gap_closure mode)
   592→- [ ] If issues: s2r-plan-checker verifies fix plans
   593→- [ ] If issues: revision loop until plans pass (max 3 iterations)
   594→- [ ] Ready for `/s2r:execute-phase --gaps-only` when complete
   595→</success_criteria>
