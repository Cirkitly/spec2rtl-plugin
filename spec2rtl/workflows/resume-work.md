     1→<trigger>
     2→Use this workflow when:
     3→- Starting a new session on an existing project
     4→- User says "continue", "what's next", "where were we", "resume"
     5→- Any planning operation when .planning/ already exists
     6→- User returns after time away from project
     7→</trigger>
     8→
     9→<purpose>
    10→Instantly restore full project context so "Where were we?" has an immediate, complete answer.
    11→</purpose>
    12→
    13→<required_reading>
    14→@~/.claude/spec2rtl/references/continuation-format.md
    15→</required_reading>
    16→
    17→<process>
    18→
    19→<step name="detect_existing_project">
    20→Check if this is an existing project:
    21→
    22→```bash
    23→ls .planning/STATE.md 2>/dev/null && echo "Project exists"
    24→ls .planning/ROADMAP.md 2>/dev/null && echo "Roadmap exists"
    25→ls .planning/PROJECT.md 2>/dev/null && echo "Project file exists"
    26→```
    27→
    28→**If STATE.md exists:** Proceed to load_state
    29→**If only ROADMAP.md/PROJECT.md exist:** Offer to reconstruct STATE.md
    30→**If .planning/ doesn't exist:** This is a new project - route to /s2r:new-project
    31→</step>
    32→
    33→<step name="load_state">
    34→
    35→Read and parse STATE.md, then PROJECT.md:
    36→
    37→```bash
    38→cat .planning/STATE.md
    39→cat .planning/PROJECT.md
    40→```
    41→
    42→**From STATE.md extract:**
    43→
    44→- **Project Reference**: Core value and current focus
    45→- **Current Position**: Phase X of Y, Plan A of B, Status
    46→- **Progress**: Visual progress bar
    47→- **Recent Decisions**: Key decisions affecting current work
    48→- **Pending Todos**: Ideas captured during sessions
    49→- **Blockers/Concerns**: Issues carried forward
    50→- **Session Continuity**: Where we left off, any resume files
    51→
    52→**From PROJECT.md extract:**
    53→
    54→- **What This Is**: Current accurate description
    55→- **Requirements**: Validated, Active, Out of Scope
    56→- **Key Decisions**: Full decision log with outcomes
    57→- **Constraints**: Hard limits on implementation (Area, Power, Timing)
    58→
    59→</step>
    60→
    61→<step name="check_incomplete_work">
    62→Look for incomplete work that needs attention:
    63→
    64→```bash
    65→# Check for continue-here files (mid-plan resumption)
    66→ls .planning/phases/*/.continue-here*.md 2>/dev/null
    67→
    68→# Check for plans without summaries (incomplete execution)
    69→for plan in .planning/phases/*/*-PLAN.md; do
    70→  summary="${plan/PLAN/SUMMARY}"
    71→  [ ! -f "$summary" ] && echo "Incomplete: $plan"
    72→done 2>/dev/null
    73→
    74→# Check for interrupted agents
    75→if [ -f .planning/current-agent-id.txt ] && [ -s .planning/current-agent-id.txt ]; then
    76→  AGENT_ID=$(cat .planning/current-agent-id.txt | tr -d '\n')
    77→  echo "Interrupted agent: $AGENT_ID"
    78→fi
    79→```
    80→
    81→**If .continue-here file exists:**
    82→
    83→- This is a mid-plan resumption point
    84→- Read the file for specific resumption context
    85→- Flag: "Found mid-plan checkpoint"
    86→
    87→**If PLAN without SUMMARY exists:**
    88→
    89→- Execution was started but not completed
    90→- Flag: "Found incomplete plan execution"
    91→
    92→**If interrupted agent found:**
    93→
    94→- Subagent was spawned but session ended before completion
    95→- Read agent-history.json for task details
    96→- Flag: "Found interrupted agent"
    97→  </step>
    98→
    99→<step name="present_status">
   100→Present complete project status to user:
   101→
   102→```
   103→╔══════════════════════════════════════════════════════════════╗
   104→║  PROJECT STATUS                                               ║
   105→╠══════════════════════════════════════════════════════════════╣
   106→║  Building: [one-liner from PROJECT.md "What This Is"]         ║
   107→║                                                               ║
   108→║  Phase: [X] of [Y] - [Phase name]                            ║
   109→║  Plan:  [A] of [B] - [Status]                                ║
   110→║  Progress: [██████░░░░] XX%                                  ║
   111→║                                                               ║
   112→║  Last activity: [date] - [what happened]                     ║
   113→╚══════════════════════════════════════════════════════════════╝
   114→
   115→[If incomplete work found:]
   116→⚠️  Incomplete work detected:
   117→    - [.continue-here file or incomplete plan]
   118→
   119→[If interrupted agent found:]
   120→⚠️  Interrupted agent detected:
   121→    Agent ID: [id]
   122→    Task: [task description from agent-history.json]
   123→    Interrupted: [timestamp]
   124→
   125→    Resume with: Task tool (resume parameter with agent ID)
   126→
   127→[If pending todos exist:]
   128→📋 [N] pending todos — /s2r:check-todos to review
   129→
   130→[If blockers exist:]
   131→⚠️  Carried concerns:
   132→    - [blocker 1]
   133→    - [blocker 2]
   134→
   135→[If alignment is not ✓:]
   136→⚠️  Brief alignment: [status] - [assessment]
   137→```
   138→
   139→</step>
   140→
   141→<step name="determine_next_action">
   142→Based on project state, determine the most logical next action:
   143→
   144→**If interrupted agent exists:**
   145→→ Primary: Resume interrupted agent (Task tool with resume parameter)
   146→→ Option: Start fresh (abandon agent work)
   147→
   148→**If .continue-here file exists:**
   149→→ Primary: Resume from checkpoint
   150→→ Option: Start fresh on current plan
   151→
   152→**If incomplete plan (PLAN without SUMMARY):**
   153→→ Primary: Complete the incomplete plan
   154→→ Option: Abandon and move on
   155→
   156→**If phase in progress, all plans complete:**
   157→→ Primary: Transition to next phase
   158→→ Option: Review completed work
   159→
   160→**If phase ready to plan:**
   161→→ Check if CONTEXT.md exists for this phase:
   162→
   163→- If CONTEXT.md missing:
   164→  → Primary: Discuss phase vision (Specs, Interfaces, PPA)
   165→  → Secondary: Plan directly (skip context gathering)
   166→- If CONTEXT.md exists:
   167→  → Primary: Plan the phase
   168→  → Option: Review roadmap
   169→
   170→**If phase ready to execute:**
   171→→ Primary: Execute next plan
   172→→ Option: Review the plan first
   173→</step>
   174→
   175→<step name="offer_options">
   176→Present contextual options based on project state:
   177→
   178→```
   179→What would you like to do?
   180→
   181→[Primary action based on state - e.g.:]
   182→1. Resume interrupted agent [if interrupted agent found]
   183→   OR
   184→1. Execute phase (/s2r:execute-phase {phase})
   185→   OR
   186→1. Discuss Phase 3 context (/s2r:discuss-phase 3) [if CONTEXT.md missing]
   187→   OR
   188→1. Plan Phase 3 (/s2r:plan-phase 3) [if CONTEXT.md exists or discuss option declined]
   189→
   190→[Secondary options:]
   191→2. Review current phase status
   192→3. Check pending todos ([N] pending)
   193→4. Review brief alignment
   194→5. Something else
   195→```
   196→
   197→**Note:** When offering phase planning, check for CONTEXT.md existence first:
   198→
   199→```bash
   200→ls .planning/phases/XX-name/*-CONTEXT.md 2>/dev/null
   201→```
   202→
   203→If missing, suggest discuss-phase before plan. If exists, offer plan directly.
   204→
   205→Wait for user selection.
   206→</step>
   207→
   208→<step name="route_to_workflow">
   209→Based on user selection, route to appropriate workflow:
   210→
   211→- **Execute plan** → Show command for user to run after clearing:
   212→  ```
   213→  ---
   214→
   215→  ## ▶ Next Up
   216→
   217→  **{phase}-{plan}: [Plan Name]** — [objective from PLAN.md]
   218→
   219→  `/s2r:execute-phase {phase}`
   220→
   221→  <sub>`/clear` first → fresh context window</sub>
   222→
   223→  ---
   224→  ```
   225→- **Plan phase** → Show command for user to run after clearing:
   226→  ```
   227→  ---
   228→
   229→  ## ▶ Next Up
   230→
   231→  **Phase [N]: [Name]** — [Goal from ROADMAP.md]
   232→
   233→  `/s2r:plan-phase [phase-number]`
   234→
   235→  <sub>`/clear` first → fresh context window</sub>
   236→
   237→  ---
   238→
   239→  **Also available:**
   240→  - `/s2r:discuss-phase [N]` — gather hardware specs first
   241→  - `/s2r:research-phase [N]` — investigate IPs and protocols
   242→
   243→  ---
   244→  ```
   245→- **Transition** → ./transition.md
   246→- **Check todos** → Read .planning/todos/pending/, present summary
   247→- **Review alignment** → Read PROJECT.md, compare to current state
   248→- **Something else** → Ask what they need
   249→</step>
   250→
   251→<step name="update_session">
   252→Before proceeding to routed workflow, update session continuity:
   253→
   254→Update STATE.md:
   255→
   256→```markdown
   257→## Session Continuity
   258→
   259→Last session: [now]
   260→Stopped at: Session resumed, proceeding to [action]
   261→Resume file: [updated if applicable]
   262→```
   263→
   264→This ensures if session ends unexpectedly, next resume knows the state.
   265→</step>
   266→
   267→</process>
   268→
   269→<reconstruction>
   270→If STATE.md is missing but other artifacts exist:
   271→
   272→"STATE.md missing. Reconstructing from artifacts..."
   273→
   274→1. Read PROJECT.md → Extract "What This Is" and Core Value
   275→2. Read ROADMAP.md → Determine phases, find current position
   276→3. Scan \*-SUMMARY.md files → Extract decisions, concerns
   277→4. Count pending todos in .planning/todos/pending/
   278→5. Check for .continue-here files → Session continuity
   279→
   280→Reconstruct and write STATE.md, then proceed normally.
   281→
   282→This handles cases where:
   283→
   284→- Project predates STATE.md introduction
   285→- File was accidentally deleted
   286→- Cloning repo without full .planning/ state
   287→  </reconstruction>
   288→
   289→<quick_resume>
   290→If user says "continue" or "go":
   291→- Load state silently
   292→- Determine primary action
   293→- Execute immediately without presenting options
   294→
   295→"Continuing from [state]... [action]"
   296→</quick_resume>
   297→
   298→<success_criteria>
   299→Resume is complete when:
   300→
   301→- [ ] STATE.md loaded (or reconstructed)
   302→- [ ] Incomplete work detected and flagged
   303→- [ ] Clear status presented to user
   304→- [ ] Contextual next actions offered
   305→- [ ] User knows exactly where project stands
   306→- [ ] Session continuity updated
   307→      </success_criteria>
   308→