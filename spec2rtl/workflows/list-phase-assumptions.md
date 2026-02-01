     1→<purpose>
     2→Surface Claude's assumptions about a phase before planning, enabling users to correct misconceptions early.
     3→
     4→Key difference from discuss-phase: This is ANALYSIS of what Claude thinks, not INTAKE of what user knows. No file output - purely conversational to prompt discussion.
     5→</purpose>
     6→
     7→<process>
     8→
     9→<step name="validate_phase" priority="first">
    10→Phase number: $ARGUMENTS (required)
    11→
    12→**If argument missing:**
    13→
    14→```
    15→Error: Phase number required.
    16→
    17→Usage: /s2r:list-phase-assumptions [phase-number]
    18→Example: /s2r:list-phase-assumptions 3
    19→```
    20→
    21→Exit workflow.
    22→
    23→**If argument provided:**
    24→Validate phase exists in roadmap:
    25→
    26→```bash
    27→cat .planning/ROADMAP.md | grep -i "Phase ${PHASE}"
    28→```
    29→
    30→**If phase not found:**
    31→
    32→```
    33→Error: Phase ${PHASE} not found in roadmap.
    34→
    35→Available phases:
    36→[list phases from roadmap]
    37→```
    38→
    39→Exit workflow.
    40→
    41→**If phase found:**
    42→Parse phase details from roadmap:
    43→
    44→- Phase number
    45→- Phase name
    46→- Phase description/goal
    47→- Any scope details mentioned
    48→
    49→Continue to analyze_phase.
    50→</step>
    51→
    52→<step name="analyze_phase">
    53→Based on roadmap description and project context, identify assumptions across five hardware domains:
    54→
    55→**1. Micro-Architecture:**
    56→What internal structure does Claude assume?
    57→- "I'd use a 3-stage pipeline because..."
    58→- "I'd implement a Round-Robin arbiter..."
    59→- "I'd use a Finite State Machine (Mealy) for control..."
    60→
    61→**2. RTL Build Order:**
    62→What sub-modules would Claude build first?
    63→- "I'd start with the leaf cells (FIFO, Counters)..."
    64→- "Then build the datapath..."
    65→- "Finally integrate into the top-level..."
    66→
    67→**3. Block Boundaries:**
    68→What logic is inside vs outside this module?
    69→- "This phase includes: Decoder, ALU, RegFile"
    70→- "This phase does NOT include: Instruction Cache, Memory Controller"
    71→- "Boundary ambiguities: Interrupt handling logic?"
    72→
    73→**4. PPA & CDC Risks:**
    74→Where does Claude expect timing/area/clocking challenges?
    75→- "The critical path will likely be the multiplier..."
    76→- "CDC issues possible on the async reset interface..."
    77→- "Area might be tight if we unroll loops..."
    78→
    79→**5. IP & Dependencies:**
    80→What IPs or standards does Claude assume?
    81→- "Assumes AXI4-Lite interface from previous phase"
    82→- "Requires Xilinx FIFO IP (or open source equivalent?)"
    83→- "Feeds into the DMA controller"
    84→
    85→Be honest about uncertainty. Mark assumptions with confidence levels:
    86→- "Fairly confident: ..." (clear from roadmap)
    87→- "Assuming: ..." (reasonable inference)
    88→- "Unclear: ..." (could go multiple ways)
    89→</step>
    90→
    91→<step name="present_assumptions">
    92→Present assumptions in a clear, scannable format:
    93→
    94→```
    95→## My Assumptions for Phase ${PHASE}: ${PHASE_NAME}
    96→
    97→### Micro-Architecture
    98→[List assumptions about internal structure]
    99→
   100→### RTL Build Order
   101→[List assumptions about implementation sequencing]
   102→
   103→### Block Boundaries
   104→**In scope:** [what's included]
   105→**Out of scope:** [what's excluded]
   106→**Ambiguous:** [what could go either way]
   107→
   108→### PPA & CDC Risks
   109→[List anticipated timing/area/clock challenges]
   110→
   111→### IP & Dependencies
   112→**From prior phases:** [what's needed]
   113→**External IPs:** [third-party needs]
   114→**Feeds into:** [what future phases need from this]
   115→
   116→---
   117→
   118→**What do you think?**
   119→
   120→Are these assumptions accurate? Let me know:
   121→- What I got right
   122→- What I got wrong
   123→- What I'm missing
   124→```
   125→
   126→Wait for user response.
   127→</step>
   128→
   129→<step name="gather_feedback">
   130→**If user provides corrections:**
   131→
   132→Acknowledge the corrections:
   133→
   134→```
   135→Key corrections:
   136→- [correction 1]
   137→- [correction 2]
   138→
   139→This changes my understanding significantly. [Summarize new understanding]
   140→```
   141→
   142→**If user confirms assumptions:**
   143→
   144→```
   145→Assumptions validated.
   146→```
   147→
   148→Continue to offer_next.
   149→</step>
   150→
   151→<step name="offer_next">
   152→Present next steps:
   153→
   154→```
   155→What's next?
   156→1. Discuss context (/s2r:discuss-phase ${PHASE}) - Let me ask you questions to build comprehensive specs
   157→2. Plan this phase (/s2r:plan-phase ${PHASE}) - Create detailed RTL implementation plans
   158→3. Re-examine assumptions - I'll analyze again with your corrections
   159→4. Done for now
   160→```
   161→
   162→Wait for user selection.
   163→
   164→If "Discuss context": Note that CONTEXT.md will incorporate any corrections discussed here
   165→If "Plan this phase": Proceed knowing assumptions are understood
   166→If "Re-examine": Return to analyze_phase with updated understanding
   167→</step>
   168→
   169→</process>
   170→
   171→<success_criteria>
   172→- Phase number validated against roadmap
   173→- Assumptions surfaced across five areas: micro-arch, build order, boundaries, PPA risks, dependencies
   174→- Confidence levels marked where appropriate
   175→- "What do you think?" prompt presented
   176→- User feedback acknowledged
   177→- Clear next steps offered
   178→</success_criteria>
