     1→<purpose>
     2→Execute discovery at the appropriate depth level.
     3→Produces DISCOVERY.md (for Level 2-3) that informs PLAN.md creation.
     4→
     5→Called from plan-phase.md's mandatory_discovery step with a depth parameter.
     6→
     7→NOTE: For comprehensive architectural research ("NOC vs Crossbar"), use /s2r:research-phase instead, which produces RESEARCH.md.
     8→</purpose>
     9→
    10→<depth_levels>
    11→**This workflow supports three depth levels:**
    12→
    13→| Level | Name         | Time      | Output                                       | When                                      |
    14→| ----- | ------------ | --------- | -------------------------------------------- | ----------------------------------------- |
    15→| 1     | Quick Verify | 2-5 min   | No file, proceed with verified knowledge     | Confirming spec version, EDA tool support |
    16→| 2     | Standard     | 15-30 min | DISCOVERY.md                                 | Choosing IP cores, Protocol details       |
    17→| 3     | Deep Dive    | 1+ hour   | Detailed DISCOVERY.md with PPA analysis      | Architecture decisions, Make vs Buy       |
    18→
    19→**Depth is determined by plan-phase.md before routing here.**
    20→</depth_levels>
    21→
    22→<source_hierarchy>
    23→**MANDATORY: Official Specs BEFORE WebSearch**
    24→
    25→Hardware errors are expensive. Hallucinated pinouts or timing are fatal.
    26→
    27→1. **Vendor Documentation** (Xilinx/Intel/Lattice) - Datasheets, User Guides.
    28→2. **Standard Specifications** (ARM AMBA, IEEE, MIPI, PCI-SIG).
    29→3. **WebSearch** - For errata, forums, and comparative analysis only.
    30→
    31→See ~/.claude/spec2rtl/templates/discovery.md `<discovery_protocol>` for full protocol.
    32→</source_hierarchy>
    33→
    34→<process>
    35→
    36→<step name="determine_depth">
    37→Check the depth parameter passed from plan-phase.md:
    38→- `depth=verify` → Level 1 (Quick Verification)
    39→- `depth=standard` → Level 2 (Standard Discovery)
    40→- `depth=deep` → Level 3 (Deep Dive)
    41→
    42→Route to appropriate level workflow below.
    43→</step>
    44→
    45→<step name="level_1_quick_verify">
    46→**Level 1: Quick Verification (2-5 minutes)**
    47→
    48→For: Confirming tool support, language features, or standard versions.
    49→
    50→**Process:**
    51→
    52→1. **Consult Knowledge Base / Context7:**
    53→   - "Does Vivado 2024.1 support SystemVerilog 'interface class'?"
    54→   - "What is the reset polarity for AXI4-Lite?"
    55→
    56→2. **Verify against Datasheet/User Guide if needed:**
    57→   - Search specific vendor doc (e.g., UG901).
    58→
    59→3. **Verify:**
    60→   - Feature is supported in target EDA tool.
    61→   - Spec interpretation is correct.
    62→
    63→4. **If verified:** Return to plan-phase.md with confirmation. No DISCOVERY.md needed.
    64→
    65→5. **If concerns found:** Escalate to Level 2.
    66→
    67→**Output:** Verbal confirmation to proceed, or escalation to Level 2.
    68→</step>
    69→
    70→<step name="level_2_standard">
    71→**Level 2: Standard Discovery (15-30 minutes)**
    72→
    73→For: Choosing IP cores, understanding protocol nuances, finding VIPs.
    74→
    75→**Process:**
    76→
    77→1. **Identify what to discover:**
    78→   - What IPs are available (Vendor vs Open Source)?
    79→   - What are the interface requirements (AXI, Wishbone)?
    80→   - Are there Verification IPs (VIPs) available?
    81→
    82→2. **Search Sources:**
    83→   - Vendor Catalog (Vivado IP Catalog, Quartus Platform Designer).
    84→   - GitHub/OpenCores for open source equivalents.
    85→   - Protocol Specs (AMBA AXI4, I2C, SPI).
    86→
    87→3. **WebSearch for comparisons:**
    88→   - "[IP A] vs [IP B] area utilization"
    89→   - "[Protocol] common pitfalls"
    90→   - "Verilator support for [IP]"
    91→
    92→4. **Create DISCOVERY.md** using ~/.claude/spec2rtl/templates/discovery.md structure:
    93→   - Selected IP/Approach.
    94→   - Interface definition.
    95→   - Resource estimates (LUTs/FFs) if available.
    96→   - Confidence level.
    97→
    98→5. Return to plan-phase.md.
    99→
   100→**Output:** `.planning/phases/XX-name/DISCOVERY.md`
   101→</step>
   102→
   103→<step name="level_3_deep_dive">
   104→**Level 3: Deep Dive (1+ hour)**
   105→
   106→For: Architectural decisions, PPA trade-offs, Make vs Buy.
   107→
   108→**Process:**
   109→
   110→1. **Scope the discovery** using ~/.claude/spec2rtl/templates/discovery.md:
   111→   - Define PPA targets (Power, Performance, Area).
   112→   - Define list of candidate architectures.
   113→
   114→2. **Deep Specification Analysis:**
   115→   - Read full Interface Specs.
   116→   - Read Architecture Manuals.
   117→
   118→3. **Ecosystem & Errata Search:**
   119→   - Search for silicon errata or known bugs.
   120→   - Look for verification challenges (e.g., "Hard to close timing on X").
   121→
   122→4. **Cross-verify ALL findings:**
   123→   - Don't trust blog posts for timing data; check reports.
   124→   - Verify license compatibility for "Open Source" IPs.
   125→
   126→5. **Create comprehensive DISCOVERY.md:**
   127→   - PPA Comparison Table.
   128→   - Risk Assessment (Timing, Area, Complexity).
   129→   - Recommended Architecture.
   130→   - If LOW confidence on critical spec → add validation checkpoints.
   131→
   132→6. **Confidence gate:** If overall confidence is LOW, present options before proceeding.
   133→
   134→7. Return to plan-phase.md.
   135→
   136→**Output:** `.planning/phases/XX-name/DISCOVERY.md` (comprehensive)
   137→</step>
   138→
   139→<step name="identify_unknowns">
   140→**For Level 2-3:** Define what we need to learn.
   141→
   142→Ask: What do we need to learn before we can plan this RTL phase?
   143→
   144→- Bus Protocol details?
   145→- Reset/Clocking topology?
   146→- Third-party IP licensing/availability?
   147→- EDA tool limitations?
   148→  </step>
   149→
   150→<step name="create_discovery_scope">
   151→Use ~/.claude/spec2rtl/templates/discovery.md.
   152→
   153→Include:
   154→- Clear discovery objective
   155→- Scoped include/exclude lists
   156→- PPA goals (if relevant)
   157→- Output structure for DISCOVERY.md
   158→  </step>
   159→
   160→<step name="execute_discovery">
   161→Run the discovery:
   162→- Use web search for current IP catalogs/Docs.
   163→- Use Context7 for standard specs (AXI, IEEE 1800).
   164→- Structure findings per template.
   165→</step>
   166→
   167→<step name="create_discovery_output">
   168→Write `.planning/phases/XX-name/DISCOVERY.md`:
   169→- Summary with recommendation
   170→- Interface definitions
   171→- PPA estimates
   172→- Metadata (confidence, dependencies, open questions, assumptions)
   173→</step>
   174→
   175→<step name="confidence_gate">
   176→After creating DISCOVERY.md, check confidence level.
   177→
   178→If confidence is LOW:
   179→Use AskUserQuestion:
   180→- header: "Low Confidence"
   181→- question: "Discovery confidence is LOW: [reason]. How would you like to proceed?"
   182→- options:
   183→  - "Dig deeper" - Do more research before planning
   184→  - "Proceed anyway" - Accept uncertainty, plan with caveats
   185→  - "Pause" - I need to think about this
   186→
   187→If confidence is MEDIUM:
   188→Inline: "Discovery complete (medium confidence). [brief reason]. Proceed to planning?"
   189→
   190→If confidence is HIGH:
   191→Proceed directly, just note: "Discovery complete (high confidence)."
   192→</step>
   193→
   194→<step name="open_questions_gate">
   195→If DISCOVERY.md has open_questions:
   196→
   197→Present them inline:
   198→"Open questions from discovery:
   199→
   200→- [Question 1]
   201→- [Question 2]
   202→
   203→These may affect implementation. Acknowledge and proceed? (yes / address first)"
   204→
   205→If "address first": Gather user input on questions, update discovery.
   206→</step>
   207→
   208→<step name="offer_next">
   209→```
   210→Discovery complete: .planning/phases/XX-name/DISCOVERY.md
   211→Recommendation: [one-liner]
   212→Confidence: [level]
   213→
   214→What's next?
   215→
   216→1. Discuss phase context (/s2r:discuss-phase [current-phase])
   217→2. Create phase plan (/s2r:plan-phase [current-phase])
   218→3. Refine discovery (dig deeper)
   219→4. Review discovery
   220→```
   221→
   222→NOTE: DISCOVERY.md is NOT committed separately. It will be committed with phase completion.
   223→</step>
   224→
   225→</process>
   226→
   227→<success_criteria>
   228→**Level 1 (Quick Verify):**
   229→- Knowledge Base / Specs consulted.
   230→- Tool/Language support verified.
   231→- Verbal confirmation to proceed (no files).
   232→
   233→**Level 2 (Standard):**
   234→- IP options evaluated (Vendor vs Open).
   235→- Interfaces/Protocols defined.
   236→- DISCOVERY.md created with recommendation.
   237→- Confidence level MEDIUM or higher.
   238→
   239→**Level 3 (Deep Dive):**
   240→- Discovery scope defined.
   241→- Specs & Architecture manuals exhaustively consulted.
   242→- PPA analysis performed.
   243→- DISCOVERY.md created with comprehensive analysis.
   244→- If LOW confidence findings → validation checkpoints defined.
   245→- Confidence gate passed.
   246→</success_criteria>
