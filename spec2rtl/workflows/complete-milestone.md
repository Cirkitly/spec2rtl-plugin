     1→<purpose>
     2→
     3→Mark a shipped version (v1.0, v1.1, v2.0) as complete. This creates a historical record in MILESTONES.md, performs full PROJECT.md evolution review, reorganizes ROADMAP.md with milestone groupings, and tags the release in git.
     4→
     5→This is the ritual that separates "development" from "shipped" (Tape-out / FPGA Release).
     6→
     7→</purpose>
     8→
     9→<required_reading>
    10→
    11→**Read these files NOW:**
    12→
    13→1. templates/milestone.md
    14→2. templates/milestone-archive.md
    15→3. `.planning/ROADMAP.md`
    16→4. `.planning/REQUIREMENTS.md`
    17→5. `.planning/PROJECT.md`
    18→
    19→</required_reading>
    20→
    21→<archival_behavior>
    22→
    23→When a milestone completes, this workflow:
    24→
    25→1. Extracts full milestone details to `.planning/milestones/v[X.Y]-ROADMAP.md`
    26→2. Archives requirements to `.planning/milestones/v[X.Y]-REQUIREMENTS.md`
    27→3. Updates ROADMAP.md to replace milestone details with one-line summary
    28→4. Deletes REQUIREMENTS.md (fresh one created for next milestone)
    29→5. Performs full PROJECT.md evolution review
    30→6. Offers to create next milestone inline
    31→
    32→**Context Efficiency:** Archives keep ROADMAP.md constant-size and REQUIREMENTS.md milestone-scoped.
    33→
    34→**Archive Format:**
    35→
    36→**ROADMAP archive** uses `templates/milestone-archive.md` template with:
    37→- Milestone header (status, phases, date)
    38→- Full phase details from roadmap
    39→- Milestone summary (decisions, issues, technical debt)
    40→
    41→**REQUIREMENTS archive** contains:
    42→- All v1 requirements marked complete with outcomes
    43→- Traceability table with final status
    44→- Notes on any requirements that changed during milestone
    45→
    46→</archival_behavior>
    47→
    48→<process>
    49→
    50→<step name="verify_readiness">
    51→
    52→Check if milestone is truly complete:
    53→
    54→```bash
    55→cat .planning/ROADMAP.md
    56→ls .planning/phases/*/SUMMARY.md 2>/dev/null | wc -l
    57→```
    58→
    59→**Questions to ask:**
    60→
    61→- Which phases belong to this milestone?
    62→- Are all those phases complete (all plans have summaries)?
    63→- Has the work been verified (Sim/FPGA)?
    64→- Is this ready to ship/tag?
    65→
    66→Present:
    67→
    68→```
    69→Milestone: [Name from user, e.g., "v1.0 FPGA Image"]
    70→
    71→Appears to include:
    72→- Phase 1: Architecture (2/2 plans complete)
    73→- Phase 2: RTL Implementation (2/2 plans complete)
    74→- Phase 3: Verification (3/3 plans complete)
    75→- Phase 4: Synthesis/Timing (1/1 plan complete)
    76→
    77→Total: 4 phases, 8 plans, all complete
    78→```
    79→
    80→<config-check>
    81→
    82→```bash
    83→cat .planning/config.json 2>/dev/null
    84→```
    85→
    86→</config-check>
    87→
    88→<if mode="yolo">
    89→
    90→```
    91→⚡ Auto-approved: Milestone scope verification
    92→
    93→[Show breakdown summary without prompting]
    94→
    95→Proceeding to stats gathering...
    96→```
    97→
    98→Proceed directly to gather_stats step.
    99→
   100→</if>
   101→
   102→<if mode="interactive" OR="custom with gates.confirm_milestone_scope true">
   103→
   104→```
   105→Ready to mark this milestone as shipped?
   106→(yes / wait / adjust scope)
   107→```
   108→
   109→Wait for confirmation.
   110→
   111→If "adjust scope": Ask which phases should be included.
   112→If "wait": Stop, user will return when ready.
   113→
   114→</if>
   115→
   116→</step>
   117→
   118→<step name="gather_stats">
   119→
   120→Calculate milestone statistics:
   121→
   122→```bash
   123→# Count phases and plans in milestone
   124→# (user specified or detected from roadmap)
   125→
   126→# Find git range
   127→git log --oneline --grep="feat(" | head -20
   128→
   129→# Count files modified in range
   130→git diff --stat FIRST_COMMIT..LAST_COMMIT | tail -1
   131→
   132→# Count LOC (Hardware Languages)
   133→find . -name "*.sv" -o -name "*.v" -o -name "*.vh" -o -name "*.vhd" -o -name "*.tcl" -o -name "*.cpp" | xargs wc -l 2>/dev/null
   134→
   135→# Calculate timeline
   136→git log --format="%ai" FIRST_COMMIT | tail -1  # Start date
   137→git log --format="%ai" LAST_COMMIT | head -1   # End date
   138→```
   139→
   140→Present summary:
   141→
   142→```
   143→Milestone Stats:
   144→- Phases: [X-Y]
   145→- Plans: [Z] total
   146→- Tasks: [N] total (estimated from phase summaries)
   147→- Files modified: [M]
   148→- Lines of code: [LOC] [language]
   149→- Timeline: [Days] days ([Start] → [End])
   150→- Git range: feat(XX-XX) → feat(YY-YY)
   151→```
   152→
   153→</step>
   154→
   155→<step name="extract_accomplishments">
   156→
   157→Read all phase SUMMARY.md files in milestone range:
   158→
   159→```bash
   160→cat .planning/phases/01-*/01-*-SUMMARY.md
   161→cat .planning/phases/02-*/02-*-SUMMARY.md
   162→# ... for each phase in milestone
   163→```
   164→
   165→From summaries, extract 4-6 key accomplishments (RTL modules, Timing closure, Verification metrics).
   166→
   167→Present:
   168→
   169→```
   170→Key accomplishments for this milestone:
   171→1. [Achievement from phase 1]
   172→2. [Achievement from phase 2]
   173→3. [Achievement from phase 3]
   174→4. [Achievement from phase 4]
   175→5. [Achievement from phase 5]
   176→```
   177→
   178→</step>
   179→
   180→<step name="create_milestone_entry">
   181→
   182→Create or update `.planning/MILESTONES.md`.
   183→
   184→If file doesn't exist:
   185→
   186→```markdown
   187→# Project Milestones: [Project Name from PROJECT.md]
   188→
   189→[New entry]
   190→```
   191→
   192→If exists, prepend new entry (reverse chronological order).
   193→
   194→Use template from `templates/milestone.md`:
   195→
   196→```markdown
   197→## v[Version] [Name] (Shipped: YYYY-MM-DD)
   198→
   199→**Delivered:** [One sentence from user]
   200→
   201→**Phases completed:** [X-Y] ([Z] plans total)
   202→
   203→**Key accomplishments:**
   204→
   205→- [List from previous step]
   206→
   207→**Stats:**
   208→
   209→- [Files] files created/modified
   210→- [LOC] lines of [language]
   211→- [Phases] phases, [Plans] plans, [Tasks] tasks
   212→- [Days] days from [start milestone or start project] to ship
   213→
   214→**Git range:** `feat(XX-XX)` → `feat(YY-YY)`
   215→
   216→**What's next:** [Ask user: what's the next goal?]
   217→
   218→---
   219→```
   220→
   221→</step>
   222→
   223→<step name="evolve_project_full_review">
   224→
   225→Perform full PROJECT.md evolution review at milestone completion.
   226→
   227→**Read all phase summaries in this milestone:**
   228→
   229→```bash
   230→cat .planning/phases/*-*/*-SUMMARY.md
   231→```
   232→
   233→**Full review checklist:**
   234→
   235→1. **"What This Is" accuracy:**
   236→   - Read current description
   237→   - Compare to what was actually built
   238→   - Update if the product has meaningfully changed
   239→
   240→2. **Core Value check:**
   241→   - Is the stated core value still the right priority?
   242→   - Did shipping reveal a different core value?
   243→   - Update if the ONE thing has shifted
   244→
   245→3. **Requirements audit:**
   246→
   247→   **Validated section:**
   248→   - All Active requirements shipped in this milestone → Move to Validated
   249→   - Format: `- ✓ [Requirement] — v[X.Y]`
   250→
   251→   **Active section:**
   252→   - Remove requirements that moved to Validated
   253→   - Add any new requirements for next milestone
   254→   - Keep requirements that weren't addressed yet
   255→
   256→   **Out of Scope audit:**
   257→   - Review each item — is the reasoning still valid?
   258→   - Remove items that are no longer relevant
   259→   - Add any requirements invalidated during this milestone
   260→
   261→4. **Context update:**
   262→   - Current codebase state (LOC, tech stack)
   263→   - Lab feedback (if any)
   264→   - Known issues or technical debt to address (Timing violations, Linter warnings)
   265→
   266→5. **Key Decisions audit:**
   267→   - Extract all decisions from milestone phase summaries
   268→   - Add to Key Decisions table with outcomes where known
   269→   - Mark ✓ Good, ⚠️ Revisit, or — Pending for each
   270→
   271→6. **Constraints check:**
   272→   - Any constraints that changed during development? (Area budget, Power budget)
   273→   - Update as needed
   274→
   275→**Update PROJECT.md:**
   276→
   277→Make all edits inline. Update "Last updated" footer:
   278→
   279→```markdown
   280→---
   281→*Last updated: [date] after v[X.Y] milestone*
   282→```
   283→
   284→**Example full evolution (v1.0 → v1.1 prep):**
   285→
   286→Before:
   287→
   288→```markdown
   289→## What This Is
   290→
   291→A RISC-V Core implementation in SystemVerilog.
   292→
   293→## Core Value
   294→
   295→Cycle-accurate execution compliant with RV32I.
   296→
   297→## Requirements
   298→
   299→### Validated
   300→
   301→(None yet — ship to validate)
   302→
   303→### Active
   304→
   305→- [ ] ALU implementation
   306→- [ ] Register File (2R1W)
   307→- [ ] Decode Stage
   308→- [ ] AXI4-Lite Interface
   309→
   310→### Out of Scope
   311→
   312→- Floating Point Unit (F)
   313→- Caches
   314→```
   315→
   316→After v1.0:
   317→
   318→```markdown
   319→## What This Is
   320→
   321→A verified RISC-V Core (RV32I) with AXI4-Lite interface.
   322→
   323→## Core Value
   324→
   325→Cycle-accurate execution compliant with RV32I.
   326→
   327→## Requirements
   328→
   329→### Validated
   330→
   331→- ✓ ALU implementation — v1.0
   332→- ✓ Register File (2R1W) — v1.0
   333→- ✓ Decode Stage — v1.0
   334→- ✓ AXI4-Lite Interface — v1.0
   335→
   336→### Active
   337→
   338→- [ ] Branch Prediction
   339→- [ ] Instruction Cache
   340→- [ ] Data Cache
   341→
   342→### Out of Scope
   343→
   344→- Floating Point Unit (F)
   345→- Multi-core support
   346→
   347→## Context
   348→
   349→Shipped v1.0 with 1,500 LOC SystemVerilog.
   350→Tech stack: Verilator, Vivado.
   351→Verified against riscv-tests.
   352→```
   353→
   354→**Step complete when:**
   355→
   356→- [ ] "What This Is" reviewed and updated if needed
   357→- [ ] Core Value verified as still correct
   358→- [ ] All shipped requirements moved to Validated
   359→- [ ] New requirements added to Active for next milestone
   360→- [ ] Out of Scope reasoning audited
   361→- [ ] Context updated with current state
   362→- [ ] All milestone decisions added to Key Decisions
   363→- [ ] "Last updated" footer reflects milestone completion
   364→
   365→</step>
   366→
   367→<step name="reorganize_roadmap">
   368→
   369→Update `.planning/ROADMAP.md` to group completed milestone phases.
   370→
   371→Add milestone headers and collapse completed work:
   372→
   373→```markdown
   374→# Roadmap: [Project Name]
   375→
   376→## Milestones
   377→
   378→- ✅ **v1.0 MVP** — Phases 1-4 (shipped YYYY-MM-DD)
   379→- 🚧 **v1.1 Performance** — Phases 5-6 (in progress)
   380→- 📋 **v2.0 Advanced** — Phases 7-10 (planned)
   381→
   382→## Phases
   383→
   384→<details>
   385→<summary>✅ v1.0 MVP (Phases 1-4) — SHIPPED YYYY-MM-DD</summary>
   386→
   387→- [x] Phase 1: Architecture (2/2 plans) — completed YYYY-MM-DD
   388→- [x] Phase 2: RTL (2/2 plans) — completed YYYY-MM-DD
   389→- [x] Phase 3: Verification (3/3 plans) — completed YYYY-MM-DD
   390→- [x] Phase 4: Synthesis (1/1 plan) — completed YYYY-MM-DD
   391→
   392→</details>
   393→
   394→### 🚧 v[Next] [Name] (In Progress / Planned)
   395→
   396→- [ ] Phase 5: [Name] ([N] plans)
   397→- [ ] Phase 6: [Name] ([N] plans)
   398→
   399→## Progress
   400→
   401→| Phase             | Milestone | Plans Complete | Status      | Completed  |
   402→| ----------------- | --------- | -------------- | ----------- | ---------- |
   403→| 1. Architecture   | v1.0      | 2/2            | Complete    | YYYY-MM-DD |
   404→| 2. RTL            | v1.0      | 2/2            | Complete    | YYYY-MM-DD |
   405→| 3. Verification   | v1.0      | 3/3            | Complete    | YYYY-MM-DD |
   406→| 4. Synthesis      | v1.0      | 1/1            | Complete    | YYYY-MM-DD |
   407→| 5. Caches         | v1.1      | 0/1            | Not started | -          |
   408→| 6. Predictor      | v1.1      | 0/2            | Not started | -          |
   409→```
   410→
   411→</step>
   412→
   413→<step name="archive_milestone">
   414→
   415→Extract completed milestone details and create archive file.
   416→
   417→**Process:**
   418→
   419→1. Create archive file path: `.planning/milestones/v[X.Y]-ROADMAP.md`
   420→
   421→2. Read `~/.claude/spec2rtl/templates/milestone-archive.md` template
   422→
   423→3. Extract data from current ROADMAP.md:
   424→   - All phases belonging to this milestone (by phase number range)
   425→   - Full phase details (goals, plans, dependencies, status)
   426→   - Phase plan lists with completion checkmarks
   427→
   428→4. Extract data from PROJECT.md:
   429→   - Key decisions made during this milestone
   430→   - Requirements that were validated
   431→
   432→5. Fill template {{PLACEHOLDERS}}:
   433→   - {{VERSION}} — Milestone version (e.g., "1.0")
   434→   - {{MILESTONE_NAME}} — From ROADMAP.md milestone header
   435→   - {{DATE}} — Today's date
   436→   - {{PHASE_START}} — First phase number in milestone
   437→   - {{PHASE_END}} — Last phase number in milestone
   438→   - {{TOTAL_PLANS}} — Count of all plans in milestone
   439→   - {{MILESTONE_DESCRIPTION}} — From ROADMAP.md overview
   440→   - {{PHASES_SECTION}} — Full phase details extracted
   441→   - {{DECISIONS_FROM_PROJECT}} — Key decisions from PROJECT.md
   442→   - {{ISSUES_RESOLVED_DURING_MILESTONE}} — From summaries
   443→
   444→6. Write filled template to `.planning/milestones/v[X.Y]-ROADMAP.md`
   445→
   446→7. Delete ROADMAP.md (fresh one created for next milestone):
   447→   ```bash
   448→   rm .planning/ROADMAP.md
   449→   ```
   450→
   451→8. Verify archive exists:
   452→   ```bash
   453→   ls .planning/milestones/v[X.Y]-ROADMAP.md
   454→   ```
   455→
   456→9. Confirm roadmap archive complete:
   457→
   458→   ```
   459→   ✅ v[X.Y] roadmap archived to milestones/v[X.Y]-ROADMAP.md
   460→   ✅ ROADMAP.md deleted (fresh one for next milestone)
   461→   ```
   462→
   463→**Note:** Phase directories (`.planning/phases/`) are NOT deleted. They accumulate across milestones as the raw execution history. Phase numbering continues (v1.0 phases 1-4, v1.1 phases 5-8, etc.).
   464→
   465→</step>
   466→
   467→<step name="archive_requirements">
   468→
   469→Archive requirements and prepare for fresh requirements in next milestone.
   470→
   471→**Process:**
   472→
   473→1. Read current REQUIREMENTS.md:
   474→   ```bash
   475→   cat .planning/REQUIREMENTS.md
   476→   ```
   477→
   478→2. Create archive file: `.planning/milestones/v[X.Y]-REQUIREMENTS.md`
   479→
   480→3. Transform requirements for archive:
   481→   - Mark all v1 requirements as `[x]` complete
   482→   - Add outcome notes where relevant (validated, adjusted, dropped)
   483→   - Update traceability table status to "Complete" for all shipped requirements
   484→   - Add "Milestone Summary" section with:
   485→     - Total requirements shipped
   486→     - Any requirements that changed scope during milestone
   487→     - Any requirements dropped and why
   488→
   489→4. Write archive file with header:
   490→   ```markdown
   491→   # Requirements Archive: v[X.Y] [Milestone Name]
   492→
   493→   **Archived:** [DATE]
   494→   **Status:** ✅ SHIPPED
   495→
   496→   This is the archived requirements specification for v[X.Y].
   497→   For current requirements, see `.planning/REQUIREMENTS.md` (created for next milestone).
   498→
   499→   ---
   500→
   501→   [Full REQUIREMENTS.md content with checkboxes marked complete]
   502→
   503→   ---
   504→
   505→   ## Milestone Summary
   506→
   507→   **Shipped:** [X] of [Y] v1 requirements
   508→   **Adjusted:** [list any requirements that changed during implementation]
   509→   **Dropped:** [list any requirements removed and why]
   510→
   511→   ---
   512→   *Archived: [DATE] as part of v[X.Y] milestone completion*
   513→   ```
   514→
   515→5. Delete original REQUIREMENTS.md:
   516→   ```bash
   517→   rm .planning/REQUIREMENTS.md
   518→   ```
   519→
   520→6. Confirm:
   521→   ```
   522→   ✅ Requirements archived to milestones/v[X.Y]-REQUIREMENTS.md
   523→   ✅ REQUIREMENTS.md deleted (fresh one needed for next milestone)
   524→   ```
   525→
   526→**Important:** The next milestone workflow starts with `/s2r:new-milestone` which includes requirements definition. PROJECT.md's Validated section carries the cumulative record across milestones.
   527→
   528→</step>
   529→
   530→<step name="archive_audit">
   531→
   532→Move the milestone audit file to the archive (if it exists):
   533→
   534→```bash
   535→# Move audit to milestones folder (if exists)
   536→[ -f .planning/v[X.Y]-MILESTONE-AUDIT.md ] && mv .planning/v[X.Y]-MILESTONE-AUDIT.md .planning/milestones/
   537→```
   538→
   539→Confirm:
   540→```
   541→✅ Audit archived to milestones/v[X.Y]-MILESTONE-AUDIT.md
   542→```
   543→
   544→(Skip silently if no audit file exists — audit is optional)
   545→
   546→</step>
   547→
   548→<step name="update_state">
   549→
   550→Update STATE.md to reflect milestone completion.
   551→
   552→**Project Reference:**
   553→
   554→```markdown
   555→## Project Reference
   556→
   557→See: .planning/PROJECT.md (updated [today])
   558→
   559→**Core value:** [Current core value from PROJECT.md]
   560→**Current focus:** [Next milestone or "Planning next milestone"]
   561→```
   562→
   563→**Current Position:**
   564→
   565→```markdown
   566→Phase: [Next phase] of [Total] ([Phase name])
   567→Plan: Not started
   568→Status: Ready to plan
   569→Last activity: [today] — v[X.Y] milestone complete
   570→
   571→Progress: [updated progress bar]
   572→```
   573→
   574→**Accumulated Context:**
   575→
   576→- Clear decisions summary (full log in PROJECT.md)
   577→- Clear resolved blockers
   578→- Keep open blockers for next milestone
   579→
   580→</step>
   581→
   582→<step name="handle_branches">
   583→
   584→Check if branching was used and offer merge options.
   585→
   586→**Check branching strategy:**
   587→
   588→```bash
   589→# Get branching strategy from config
   590→BRANCHING_STRATEGY=$(cat .planning/config.json 2>/dev/null | grep -o '"branching_strategy"[[:space:]]*:[[:space:]]*"[^"]*"' | sed 's/.*:.*"\([^"]*\)"/\1/' || echo "none")
   591→```
   592→
   593→**If strategy is "none":** Skip to git_tag step.
   594→
   595→**For "phase" strategy — find phase branches:**
   596→
   597→```bash
   598→PHASE_BRANCH_TEMPLATE=$(cat .planning/config.json 2>/dev/null | grep -o '"phase_branch_template"[[:space:]]*:[[:space:]]*"[^"]*"' | sed 's/.*:.*"\([^"]*\)"/\1/' || echo "gsd/phase-{phase}-{slug}")
   599→
   600→# Extract prefix from template (before first variable)
   601→BRANCH_PREFIX=$(echo "$PHASE_BRANCH_TEMPLATE" | sed 's/{.*//')
   602→
   603→# Find all phase branches for this milestone
   604→PHASE_BRANCHES=$(git branch --list "${BRANCH_PREFIX}*" 2>/dev/null | sed 's/^\*//' | tr -d ' ')
   605→```
   606→
   607→**For "milestone" strategy — find milestone branch:**
   608→
   609→```bash
   610→MILESTONE_BRANCH_TEMPLATE=$(cat .planning/config.json 2>/dev/null | grep -o '"milestone_branch_template"[[:space:]]*:[[:space:]]*"[^"]*"' | sed 's/.*:.*"\([^"]*\)"/\1/' || echo "gsd/{milestone}-{slug}")
   611→
   612→# Extract prefix from template
   613→BRANCH_PREFIX=$(echo "$MILESTONE_BRANCH_TEMPLATE" | sed 's/{.*//')
   614→
   615→# Find milestone branch
   616→MILESTONE_BRANCH=$(git branch --list "${BRANCH_PREFIX}*" 2>/dev/null | sed 's/^\*//' | tr -d ' ' | head -1)
   617→```
   618→
   619→**If no branches found:** Skip to git_tag step.
   620→
   621→**If branches exist — present merge options:**
   622→
   623→```
   624→## Git Branches Detected
   625→
   626→Branching strategy: {phase/milestone}
   627→
   628→Branches found:
   629→{list of branches}
   630→
   631→Options:
   632→1. **Merge to main** — Merge branch(es) to main
   633→2. **Delete without merging** — Branches already merged or not needed
   634→3. **Keep branches** — Leave for manual handling
   635→```
   636→
   637→Use AskUserQuestion:
   638→
   639→```
   640→AskUserQuestion([
   641→  {
   642→    question: "How should branches be handled?",
   643→    header: "Branches",
   644→    multiSelect: false,
   645→    options: [
   646→      { label: "Squash merge (Recommended)", description: "Squash all commits into one clean commit on main" },
   647→      { label: "Merge with history", description: "Preserve all individual commits (--no-ff)" },
   648→      { label: "Delete without merging", description: "Branches already merged or not needed" },
   649→      { label: "Keep branches", description: "Leave branches for manual handling later" }
   650→    ]
   651→  }
   652→])
   653→```
   654→
   655→**If "Squash merge":**
   656→
   657→```bash
   658→CURRENT_BRANCH=$(git branch --show-current)
   659→git checkout main
   660→
   661→# For phase strategy - squash merge each phase branch
   662→if [ "$BRANCHING_STRATEGY" = "phase" ]; then
   663→  for branch in $PHASE_BRANCHES; do
   664→    echo "Squash merging $branch..."
   665→    git merge --squash "$branch"
   666→    git commit -m "feat: $branch for v[X.Y]"
   667→  done
   668→fi
   669→
   670→# For milestone strategy - squash merge milestone branch
   671→if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
   672→  echo "Squash merging $MILESTONE_BRANCH..."
   673→  git merge --squash "$MILESTONE_BRANCH"
   674→  git commit -m "feat: $MILESTONE_BRANCH for v[X.Y]"
   675→fi
   676→
   677→git checkout "$CURRENT_BRANCH"
   678→```
   679→
   680→Report: "Squash merged branches to main"
   681→
   682→**If "Merge with history":**
   683→
   684→```bash
   685→CURRENT_BRANCH=$(git branch --show-current)
   686→git checkout main
   687→
   688→# For phase strategy - merge each phase branch
   689→if [ "$BRANCHING_STRATEGY" = "phase" ]; then
   690→  for branch in $PHASE_BRANCHES; do
   691→    echo "Merging $branch..."
   692→    git merge --no-ff "$branch" -m "Merge branch '$branch' for v[X.Y]"
   693→  done
   694→fi
   695→
   696→# For milestone strategy - merge milestone branch
   697→if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
   698→  echo "Merging $MILESTONE_BRANCH..."
   699→  git merge --no-ff "$MILESTONE_BRANCH" -m "Merge branch '$MILESTONE_BRANCH' for v[X.Y]"
   700→fi
   701→
   702→git checkout "$CURRENT_BRANCH"
   703→```
   704→
   705→Report: "Merged branches to main with full history"
   706→
   707→**If "Delete without merging":**
   708→
   709→```bash
   710→if [ "$BRANCHING_STRATEGY" = "phase" ]; then
   711→  for branch in $PHASE_BRANCHES; do
   712→    git branch -d "$branch" 2>/dev/null || git branch -D "$branch"
   713→  done
   714→fi
   715→
   716→if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
   717→  git branch -d "$MILESTONE_BRANCH" 2>/dev/null || git branch -D "$MILESTONE_BRANCH"
   718→fi
   719→```
   720→
   721→Report: "Deleted branches"
   722→
   723→**If "Keep branches":**
   724→
   725→Report: "Branches preserved for manual handling"
   726→
   727→</step>
   728→
   729→<step name="git_tag">
   730→
   731→Create git tag for milestone:
   732→
   733→```bash
   734→git tag -a v[X.Y] -m "$(cat <<'EOF'
   735→v[X.Y] [Name]
   736→
   737→Delivered: [One sentence]
   738→
   739→Key accomplishments:
   740→- [Item 1]
   741→- [Item 2]
   742→- [Item 3]
   743→
   744→See .planning/MILESTONES.md for full details.
   745→EOF
   746→)"
   747→```
   748→
   749→Confirm: "Tagged: v[X.Y]"
   750→
   751→Ask: "Push tag to remote? (y/n)"
   752→
   753→If yes:
   754→
   755→```bash
   756→git push origin v[X.Y]
   757→```
   758→
   759→</step>
   760→
   761→<step name="git_commit_milestone">
   762→
   763→Commit milestone completion including archive files and deletions.
   764→
   765→**Check planning config:**
   766→
   767→```bash
   768→COMMIT_PLANNING_DOCS=$(cat .planning/config.json 2>/dev/null | grep -o '"commit_docs"[[:space:]]*:[[:space:]]*[^,}]*' | grep -o 'true\|false' || echo "true")
   769→git check-ignore -q .planning 2>/dev/null && COMMIT_PLANNING_DOCS=false
   770→```
   771→
   772→**If `COMMIT_PLANNING_DOCS=false`:** Skip git operations
   773→
   774→**If `COMMIT_PLANNING_DOCS=true` (default):**
   775→
   776→```bash
   777→# Stage archive files (new)
   778→git add .planning/milestones/v[X.Y]-ROADMAP.md
   779→git add .planning/milestones/v[X.Y]-REQUIREMENTS.md
   780→git add .planning/milestones/v[X.Y]-MILESTONE-AUDIT.md 2>/dev/null || true
   781→
   782→# Stage updated files
   783→git add .planning/MILESTONES.md
   784→git add .planning/PROJECT.md
   785→git add .planning/STATE.md
   786→
   787→# Stage deletions
   788→git add -u .planning/
   789→
   790→# Commit with descriptive message
   791→git commit -m "$(cat <<'EOF'
   792→chore: complete v[X.Y] milestone
   793→
   794→Archived:
   795→- milestones/v[X.Y]-ROADMAP.md
   796→- milestones/v[X.Y]-REQUIREMENTS.md
   797→- milestones/v[X.Y]-MILESTONE-AUDIT.md (if audit was run)
   798→
   799→Deleted (fresh for next milestone):
   800→- ROADMAP.md
   801→- REQUIREMENTS.md
   802→
   803→Updated:
   804→- MILESTONES.md (new entry)
   805→- PROJECT.md (requirements → Validated)
   806→- STATE.md (reset for next milestone)
   807→
   808→Tagged: v[X.Y]
   809→EOF
   810→)"
   811→```
   812→
   813→Confirm: "Committed: chore: complete v[X.Y] milestone"
   814→
   815→</step>
   816→
   817→<step name="offer_next">
   818→
   819→```
   820→✅ Milestone v[X.Y] [Name] complete
   821→
   822→Shipped:
   823→- [N] phases ([M] plans, [P] tasks)
   824→- [One sentence of what shipped]
   825→
   826→Archived:
   827→- milestones/v[X.Y]-ROADMAP.md
   828→- milestones/v[X.Y]-REQUIREMENTS.md
   829→
   830→Summary: .planning/MILESTONES.md
   831→Tag: v[X.Y]
   832→
   833→---
   834→
   835→## ▶ Next Up
   836→
   837→**Start Next Milestone** — questioning → research → requirements → roadmap
   838→
   839→`/s2r:new-milestone`
   840→
   841→<sub>`/clear` first → fresh context window</sub>
   842→
   843→---
   844→```
   845→
   846→</step>
   847→
   848→</process>
   849→
   850→<milestone_naming>
   851→
   852→**Version conventions:**
   853→- **v1.0** — Initial MVP
   854→- **v1.1, v1.2, v1.3** — Minor updates, new features, fixes
   855→- **v2.0, v3.0** — Major rewrites, breaking changes, significant new direction
   856→
   857→**Name conventions:**
   858→- v1.0 MVP
   859→- v1.1 Timing Closure
   860→- v1.2 Interface Features
   861→- v2.0 Pipelined Arch
   862→- v2.0 Tape-out Ready
   863→
   864→Keep names short (1-2 words describing the focus).
   865→
   866→</milestone_naming>
   867→
   868→<what_qualifies>
   869→
   870→**Create milestones for:**
   871→- Tape-out / Release
   872→- FPGA Bitstream Release
   873→- Major Block Completion (e.g. CPU Core)
   874→- Before archiving planning
   875→
   876→**Don't create milestones for:**
   877→- Every phase completion (too granular)
   878→- Work in progress (wait until shipped)
   879→- Internal dev iterations (unless truly shipped internally)
   880→
   881→If uncertain, ask: "Is this deployed/usable/shipped in some form?"
   882→If yes → milestone. If no → keep working.
   883→
   884→</what_qualifies>
   885→
   886→<success_criteria>
   887→
   888→Milestone completion is successful when:
   889→
   890→- [ ] MILESTONES.md entry created with stats and accomplishments
   891→- [ ] PROJECT.md full evolution review completed
   892→- [ ] All shipped requirements moved to Validated in PROJECT.md
   893→- [ ] Key Decisions updated with outcomes
   894→- [ ] ROADMAP.md reorganized with milestone grouping
   895→- [ ] Roadmap archive created (milestones/v[X.Y]-ROADMAP.md)
   896→- [ ] Requirements archive created (milestones/v[X.Y]-REQUIREMENTS.md)
   897→- [ ] REQUIREMENTS.md deleted (fresh for next milestone)
   898→- [ ] STATE.md updated with fresh project reference
   899→- [ ] Git tag created (v[X.Y])
   900→- [ ] Milestone commit made (includes archive files and deletion)
   901→- [ ] User knows next step (/s2r:new-milestone)
   902→
   903→</success_criteria>
