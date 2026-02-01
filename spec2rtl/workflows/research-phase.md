<purpose>
Investigate "How do experts build this?" before planning.
Produces RESEARCH.md with authoritative sources (Datasheets, Specs) to guide architecture.
</purpose>

<core_principle>
**Hardware requires authoritative specs.**
Guessing a register offset or protocol handshake leads to silicon bugs.
Research must find the *exact* datasheet, the *exact* version of the AXI spec, and the *exact* FPGA primitive to use.
</core_principle>

<process>

<step name="validate_phase" priority="first">
Phase number: $ARGUMENTS (required)

Validate against ROADMAP.md.
</step>

<step name="resolve_model_profile">
Read model profile. Default: "balanced".
</step>

<step name="spawn_researcher">
Spawn `s2r-phase-researcher` agent.

**Prompt Context:**
- **Phase:** {phase_name}
- **Goal:** {phase_goal}
- **Project Context:** PPA goals, Technology Node (from PROJECT.md)
- **Constraint:** "Cite authoritative sources (Vendor Docs, IEEE Specs). Do not hallucinate register maps."

**Agent Task:**
1. Identify key technologies (e.g., "DDR4 PHY", "PCIe Gen3").
2. Search for:
   - Official Specs (JEDEC, PCI-SIG, ARM AMBA).
   - Vendor Primitives (Xilinx XPHY, Intel PHY).
   - Open Source Reference Designs (what have others done?).
   - Silicon Errata / Known Issues.
3. Synthesize into `RESEARCH.md`.
</step>

<step name="review_output">
Display summary of findings.

**Checklist:**
- [ ] Protocol versions identified (e.g., AXI4 vs AXI3)?
- [ ] Hard IP vs Soft Logic decided?
- [ ] Critical timing constraints identified?
- [ ] Pinout/IO requirements listed?

**Next Steps:**
- `/s2r:discuss-phase {N}` (Refine specs based on research)
- `/s2r:plan-phase {N}` (Start implementation)
</step>

</process>

<success_criteria>
- RESEARCH.md created in phase directory.
- Sources listed (URLs to PDFs/Docs).
- "Unknowns" reduced to actionable engineering choices.
</success_criteria>
