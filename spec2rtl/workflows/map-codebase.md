     1→<purpose>
     2→Orchestrate parallel codebase mapper agents to analyze Hardware/RTL codebase and produce structured documents in .planning/codebase/
     3→
     4→Each agent has fresh context, explores a specific focus area, and **writes documents directly**. The orchestrator only receives confirmation + line counts, then writes a summary.
     5→
     6→Output: .planning/codebase/ folder with 7 structured documents about the hardware design state.
     7→</purpose>
     8→
     9→<philosophy>
    10→**Why dedicated mapper agents:**
    11→- Fresh context per domain (no token contamination)
    12→- Agents write documents directly (no context transfer back to orchestrator)
    13→- Faster execution (agents run simultaneously)
    14→
    15→**Document quality over length:**
    16→Include enough detail to be useful as reference. Prioritize practical examples (code patterns) and file lists (.f) over arbitrary brevity.
    17→
    18→**Always include file paths:**
    19→Documents are reference material for Claude when planning/executing. Always include actual file paths formatted with backticks: `rtl/core/alu.sv`.
    20→</philosophy>
    21→
    22→<process>
    23→
    24→<step name="resolve_model_profile" priority="first">
    25→Read model profile for agent spawning:
    26→
    27→```bash
    28→MODEL_PROFILE=$(cat .planning/config.json 2>/dev/null | grep -o '"model_profile"[[:space:]]*:[[:space:]]*"[^"]*"' | grep -o '"[^"]*"$' | tr -d '"' || echo "balanced")
    29→```
    30→
    31→Default to "balanced" if not set.
    32→
    33→**Model lookup table:**
    34→
    35→| Agent | quality | balanced | budget |
    36→|-------|---------|----------|--------|
    37→| s2r-codebase-mapper | sonnet | haiku | haiku |
    38→
    39→Store resolved model for use in Task calls below.
    40→</step>
    41→
    42→<step name="check_existing">
    43→Check if .planning/codebase/ already exists:
    44→
    45→```bash
    46→ls -la .planning/codebase/ 2>/dev/null
    47→```
    48→
    49→**If exists:**
    50→
    51→```
    52→.planning/codebase/ already exists with these documents:
    53→[List files found]
    54→
    55→What's next?
    56→1. Refresh - Delete existing and remap codebase
    57→2. Update - Keep existing, only update specific documents
    58→3. Skip - Use existing codebase map as-is
    59→```
    60→
    61→Wait for user response.
    62→
    63→If "Refresh": Delete .planning/codebase/, continue to create_structure
    64→If "Update": Ask which documents to update, continue to spawn_agents (filtered)
    65→If "Skip": Exit workflow
    66→
    67→**If doesn't exist:**
    68→Continue to create_structure.
    69→</step>
    70→
    71→<step name="create_structure">
    72→Create .planning/codebase/ directory:
    73→
    74→```bash
    75→mkdir -p .planning/codebase
    76→```
    77→
    78→**Expected output files:**
    79→- STACK.md (from tech mapper)
    80→- IPS.md (from tech mapper)
    81→- HIERARCHY.md (from arch mapper)
    82→- STRUCTURE.md (from arch mapper)
    83→- CONVENTIONS.md (from quality mapper)
    84→- VERIFICATION.md (from quality mapper)
    85→- CONSTRAINTS.md (from concerns mapper)
    86→
    87→Continue to spawn_agents.
    88→</step>
    89→
    90→<step name="spawn_agents">
    91→Spawn 4 parallel s2r-codebase-mapper agents.
    92→
    93→Use Task tool with `subagent_type="s2r-codebase-mapper"`, `model="{mapper_model}"`, and `run_in_background=true` for parallel execution.
    94→
    95→**CRITICAL:** Use the dedicated `s2r-codebase-mapper` agent, NOT `Explore`. The mapper agent writes documents directly.
    96→
    97→**Agent 1: Tech Focus (EDA & IPs)**
    98→
    99→Task tool parameters:
   100→```
   101→subagent_type: "s2r-codebase-mapper"
   102→model: "{mapper_model}"
   103→run_in_background: true
   104→description: "Map EDA stack and IPs"
   105→```
   106→
   107→Prompt:
   108→```
   109→Focus: tech
   110→
   111→Analyze this codebase for EDA tools, Language versions, and IP Cores.
   112→
   113→Write these documents to .planning/codebase/:
   114→- STACK.md - Language Standards (SystemVerilog 2012, VHDL), Simulators (Verilator, VCS), Synthesis Tools (Vivado, Yosys).
   115→- IPS.md - Vendor IPs (Xilinx XCI), Third-party cores, VIPs, and external interfaces.
   116→
   117→Explore thoroughly. Write documents directly using templates. Return confirmation only.
   118→```
   119→
   120→**Agent 2: Architecture Focus (RTL Hierarchy)**
   121→
   122→Task tool parameters:
   123→```
   124→subagent_type: "s2r-codebase-mapper"
   125→model: "{mapper_model}"
   126→run_in_background: true
   127→description: "Map RTL hierarchy"
   128→```
   129→
   130→Prompt:
   131→```
   132→Focus: arch
   133→
   134→Analyze this codebase architecture, module hierarchy, and directory structure.
   135→
   136→Write these documents to .planning/codebase/:
   137→- HIERARCHY.md - Top-level modules, Bus fabric (AXI/AHB), Clock domains, Reset trees.
   138→- STRUCTURE.md - Directory layout (rtl/, tb/, sim/), file lists (.f), include paths.
   139→
   140→Explore thoroughly. Write documents directly using templates. Return confirmation only.
   141→```
   142→
   143→**Agent 3: Quality Focus (Verification)**
   144→
   145→Task tool parameters:
   146→```
   147→subagent_type: "s2r-codebase-mapper"
   148→model: "{mapper_model}"
   149→run_in_background: true
   150→description: "Map verification methodology"
   151→```
   152→
   153→Prompt:
   154→```
   155→Focus: quality
   156→
   157→Analyze this codebase for coding conventions and verification environment.
   158→
   159→Write these documents to .planning/codebase/:
   160→- CONVENTIONS.md - Naming (clk_*, *_i), Reset polarity, FSM coding style.
   161→- VERIFICATION.md - UVM/Testbench structure, Test plans, Regression scripts, Coverage goals.
   162→
   163→Explore thoroughly. Write documents directly using templates. Return confirmation only.
   164→```
   165→
   166→**Agent 4: Concerns Focus (Constraints)**
   167→
   168→Task tool parameters:
   169→```
   170→subagent_type: "s2r-codebase-mapper"
   171→model: "{mapper_model}"
   172→run_in_background: true
   173→description: "Map design constraints"
   174→```
   175→
   176→Prompt:
   177→```
   178→Focus: concerns
   179→
   180→Analyze this codebase for design constraints, CDC risks, and lint warnings.
   181→
   182→Write this document to .planning/codebase/:
   183→- CONSTRAINTS.md - Clock frequencies, Area budgets, CDC issues, Lint waivers, TODOs.
   184→
   185→Explore thoroughly. Write document directly using template. Return confirmation only.
   186→```
   187→
   188→Continue to collect_confirmations.
   189→</step>
   190→
   191→<step name="collect_confirmations">
   192→Wait for all 4 agents to complete.
   193→
   194→Read each agent's output file to collect confirmations.
   195→
   196→**Expected confirmation format from each agent:**
   197→```
   198→## Mapping Complete
   199→
   200→**Focus:** {focus}
   201→**Documents written:**
   202→- `.planning/codebase/{DOC1}.md` ({N} lines)
   203→- `.planning/codebase/{DOC2}.md` ({N} lines)
   204→
   205→Ready for orchestrator summary.
   206→```
   207→
   208→**What you receive:** Just file paths and line counts. NOT document contents.
   209→
   210→If any agent failed, note the failure and continue with successful documents.
   211→
   212→Continue to verify_output.
   213→</step>
   214→
   215→<step name="verify_output">
   216→Verify all documents created successfully:
   217→
   218→```bash
   219→ls -la .planning/codebase/
   220→wc -l .planning/codebase/*.md
   221→```
   222→
   223→**Verification checklist:**
   224→- All 7 documents exist
   225→- No empty documents (each should have >20 lines)
   226→
   227→If any documents missing or empty, note which agents may have failed.
   228→
   229→Continue to commit_codebase_map.
   230→</step>
   231→
   232→<step name="commit_codebase_map">
   233→Commit the codebase map:
   234→
   235→**Check planning config:**
   236→
   237→```bash
   238→COMMIT_PLANNING_DOCS=$(cat .planning/config.json 2>/dev/null | grep -o '"commit_docs"[[:space:]]*:[[:space:]]*[^,}]*' | grep -o 'true\|false' || echo "true")
   239→git check-ignore -q .planning 2>/dev/null && COMMIT_PLANNING_DOCS=false
   240→```
   241→
   242→**If `COMMIT_PLANNING_DOCS=false`:** Skip git operations
   243→
   244→**If `COMMIT_PLANNING_DOCS=true` (default):**
   245→
   246→```bash
   247→git add .planning/codebase/*.md
   248→git commit -m "$(cat <<'EOF'
   249→docs: map existing hardware codebase
   250→
   251→- STACK.md - EDA Tools and Languages
   252→- HIERARCHY.md - RTL Module Hierarchy
   253→- STRUCTURE.md - Directory layout
   254→- CONVENTIONS.md - RTL Coding Style
   255→- VERIFICATION.md - Testbench and UVM
   256→- IPS.md - Vendor Cores and VIPs
   257→- CONSTRAINTS.md - PPA and CDC
   258→EOF
   259→)"
   260→```
   261→
   262→Continue to offer_next.
   263→</step>
   264→
   265→<step name="offer_next">
   266→Present completion summary and next steps.
   267→
   268→**Get line counts:**
   269→```bash
   270→wc -l .planning/codebase/*.md
   271→```
   272→
   273→**Output format:**
   274→
   275→```
   276→Codebase mapping complete.
   277→
   278→Created .planning/codebase/:
   279→- STACK.md ([N] lines) - EDA Tools and Language versions
   280→- HIERARCHY.md ([N] lines) - Top-level modules and bus structure
   281→- STRUCTURE.md ([N] lines) - Directory layout and file lists
   282→- CONVENTIONS.md ([N] lines) - Naming and coding style
   283→- VERIFICATION.md ([N] lines) - Testbench structure and coverage
   284→- IPS.md ([N] lines) - Vendor IPs and VIPs
   285→- CONSTRAINTS.md ([N] lines) - Design constraints and CDC issues
   286→
   287→
   288→---
   289→
   290→## ▶ Next Up
   291→
   292→**Initialize project** — use codebase context for planning
   293→
   294→`/s2r:new-project`
   295→
   296→<sub>`/clear` first → fresh context window</sub>
   297→
   298→---
   299→
   300→**Also available:**
   301→- Re-run mapping: `/s2r:map-codebase`
   302→- Review specific file: `cat .planning/codebase/HIERARCHY.md`
   303→- Edit any document before proceeding
   304→
   305→---
   306→```
   307→
   308→End workflow.
   309→</step>
   310→
   311→</process>
   312→
   313→<success_criteria>
   314→- .planning/codebase/ directory created
   315→- 4 parallel s2r-codebase-mapper agents spawned with run_in_background=true
   316→- Agents write documents directly (orchestrator doesn't receive document contents)
   317→- Read agent output files to collect confirmations
   318→- All 7 codebase documents exist
   319→- Clear completion summary with line counts
   320→- User offered clear next steps in Spec2RTL style
   321→</success_criteria>
