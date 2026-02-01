# Discovery Template

Template for `.planning/phases/XX-name/DISCOVERY.md` - shallow research for library/IP/Option decisions.

**Purpose:** Answer "which IP/Option should we use" questions during mandatory discovery in plan-phase.

For deep ecosystem research ("how do experts build this"), use `/s2r:research-phase` which produces RESEARCH.md.

---

## File Template

```markdown
---
phase: XX-name
type: discovery
topic: [discovery-topic]
---

<session_initialization>
Before beginning discovery, verify today's date:
!`date +%Y-%m-%d`

Use this date when searching for "current" or "latest" information.
</session_initialization>

<discovery_objective>
Discover [topic] to inform [phase name] implementation.

Purpose: [What decision/implementation this enables]
Scope: [Boundaries]
Output: DISCOVERY.md with recommendation
</discovery_objective>

<discovery_scope>
<include>
- [Question to answer]
- [Area to investigate]
- [Specific comparison if needed]
</include>

<exclude>
- [Out of scope for this discovery]
- [Defer to implementation phase]
</exclude>
</discovery_scope>

<discovery_protocol>

**Source Priority:**
1. **Vendor Documentation** - Xilinx/Intel/ARM Specs (current, authoritative)
2. **Standard Specs** - AMBA (AXI/APB), IEEE, JEDEC
3. **WebSearch** - For forum discussions, known bugs, version compatibility

**Quality Checklist:**
Before completing discovery, verify:
- [ ] All claims have authoritative sources (Vendor Docs or Standards)
- [ ] Negative claims ("X is not supported") verified with official documentation
- [ ] Pinouts/Signals checked against Datasheets
- [ ] License requirements checked (Free vs Paid IP)
- [ ] Recent Errata checked for known silicon bugs

**Confidence Levels:**
- HIGH: Official Datasheet/Spec confirms
- MEDIUM: WebSearch + Official confirm
- LOW: WebSearch only (mark for validation)

</discovery_protocol>


<output_structure>
Create `.planning/phases/XX-name/DISCOVERY.md`:

```markdown
# [Topic] Discovery

## Summary
[2-3 paragraph executive summary - what was researched, what was found, what's recommended]

## Primary Recommendation
[What to do and why - be specific and actionable. e.g., "Use Xilinx AXI DMA in Simple Mode"]

## Alternatives Considered
[What else was evaluated and why not chosen]

## Key Findings

### [Category 1]
- [Finding with source URL and relevance to our case]

### [Category 2]
- [Finding with source URL and relevance]

## Implementation Details
[Relevant register offsets, signal names, parameter settings]

## Metadata

<metadata>
<confidence level="high|medium|low">
[Why this confidence level - based on source quality and verification]
</confidence>

<sources>
- [Primary authoritative sources used]
</sources>

<open_questions>
[What couldn't be determined or needs validation during implementation]
</open_questions>

<validation_checkpoints>
[If confidence is LOW or MEDIUM, list specific things to verify during implementation]
</validation_checkpoints>
</metadata>
```
</output_structure>

<success_criteria>
- All scope questions answered with authoritative sources
- Quality checklist items completed
- Clear primary recommendation
- Low-confidence findings marked with validation checkpoints
- Ready to inform PLAN.md creation
</success_criteria>

<guidelines>
**When to use discovery:**
- IP choice unclear (XDMA vs QDMA)
- Protocol version decision (AXI3 vs AXI4)
- Hard Block capabilities (Does this FPGA part have PCIe Gen4?)
- Single decision pending

**When NOT to use:**
- Established patterns (AXI-Lite for CSRs)
- Implementation details (defer to execution)
- Questions answerable from existing project context

**When to use RESEARCH.md instead:**
- Niche/complex domains (DDR4 Physics, PCIe TLP layer)
- Need ecosystem knowledge, not just IP choice
- "How do experts build this" questions
- Use `/s2r:research-phase` for these
</guidelines>
