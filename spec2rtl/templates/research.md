# Research Template

Template for `.planning/phases/XX-name/{phase}-RESEARCH.md` - comprehensive ecosystem research before planning.

**Purpose:** Document what Claude needs to know to implement a phase well - "which IP," "which Protocol version," "which EDA features."

---

## File Template

```markdown
# Phase [X]: [Name] - Research

**Researched:** [date]
**Domain:** [primary technology/problem domain]
**Confidence:** [HIGH/MEDIUM/LOW]

<research_summary>
## Summary

[2-3 paragraph executive summary]
- What was researched (IPs, Protocols, Architectures)
- Standard Industry Approach
- Key recommendations

**Primary recommendation:** [one-liner actionable guidance]
</research_summary>

<standard_stack>
## Standard Stack / IPs

The established IPs/Standards for this domain:

### Core IPs / Libraries
| Name | Version/Standard | Purpose | Why Standard |
|------|------------------|---------|--------------|
| [name] | [ver] | [what it does] | [why used] |
| [name] | [ver] | [what it does] | [why used] |

### Supporting Tools
| Tool | Feature | Purpose | Use Case |
|------|---------|---------|----------|
| [name] | [feature] | [what it does] | [when to use] |

### Alternatives Considered
| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| [standard] | [alternative] | [when alternative makes sense] |

</standard_stack>

<architecture_patterns>
## Architecture Patterns

### Recommended Hierarchy
```
rtl/
├── [module]/        # [purpose]
├── [module]/        # [purpose]
└── [module]/        # [purpose]
```

### Pattern 1: [Pattern Name]
**What:** [description]
**When to use:** [conditions]
**Example:**
```systemverilog
// [code example]
```

### Pattern 2: [Pattern Name]
**What:** [description]
**When to use:** [conditions]
**Example:**
```systemverilog
// [code example]
```

### Anti-Patterns to Avoid
- **[Anti-pattern]:** [why it's bad, what to do instead]
</architecture_patterns>

<dont_hand_roll>
## Don't Hand-Roll

Problems that look simple but have existing solutions:

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| [problem] | [custom logic] | [standard IP] | [complexity, bugs] |
| FIFO | Custom counters | Standard FIFO IP | Full/Empty logic subtlety |
| CDC | Double Flops | Vendor Macros | Constraint handling |

**Key insight:** [why custom solutions are worse in this domain]
</dont_hand_roll>

<common_pitfalls>
## Common Pitfalls

### Pitfall 1: [Name]
**What goes wrong:** [description]
**Why it happens:** [root cause]
**How to avoid:** [prevention strategy]
**Warning signs:** [how to detect early]

### Pitfall 2: [Name]
**What goes wrong:** [description]
**Why it happens:** [root cause]
**How to avoid:** [prevention strategy]
**Warning signs:** [how to detect early]
</common_pitfalls>

<code_examples>
## Code Examples

Verified patterns from official sources:

### [Common Operation 1]
```systemverilog
// Source: [Spec/Doc URL]
[code]
```

### [Common Operation 2]
```systemverilog
// Source: [Spec/Doc URL]
[code]
```
</code_examples>

<sota_updates>
## State of the Art (Current Year)

What's changed recently:

| Old Approach | Current Approach | Impact |
|--------------|------------------|--------|
| [old] | [new] | [what it means] |

**New tools/patterns to consider:**
- [Tool/Pattern]: [what it enables, when to use]
</sota_updates>

<open_questions>
## Open Questions

Things that couldn't be fully resolved:

1. **[Question]**
   - What we know: [partial info]
   - What's unclear: [the gap]
   - Recommendation: [how to handle]
</open_questions>

<sources>
## Sources

### Primary (HIGH confidence)
- [Datasheet/Spec ID] - [topics fetched]
- [Vendor Doc URL] - [what was checked]

### Secondary (MEDIUM confidence)
- [Forum/Paper] - [finding]

### Tertiary (LOW confidence)
- [WebSearch] - [finding]
</sources>

<metadata>
## Metadata

**Research scope:**
- Core technology: [what]
- Protocols: [protocols explored]
- Patterns: [patterns researched]
- Pitfalls: [areas checked]

**Confidence breakdown:**
- IPs: [HIGH/MEDIUM/LOW]
- Architecture: [HIGH/MEDIUM/LOW]
- Pitfalls: [HIGH/MEDIUM/LOW]

**Research date:** [date]
**Valid until:** [estimate]
</metadata>

---
*Phase: XX-name*
*Research completed: [date]*
*Ready for planning: [yes/no]*
```

---

## Good Example (Hardware)

```markdown
# Phase 3: PCIe DMA - Research

**Researched:** 2026-02-20
**Domain:** PCIe Gen3 x4 DMA Engine on Xilinx UltraScale+
**Confidence:** HIGH

<research_summary>
## Summary

Researched PCIe DMA implementation for Xilinx UltraScale+. Standard approach is to use the Xilinx XDMA (DMA/Bridge Subsystem for PCI Express) rather than implementing a custom Scatter-Gather engine on top of the PCIe Hard Block.

Key finding: Custom DMA engines are prone to deadlock and TLP ordering violations. XDMA provides a high-performance, validated AXI4 interface.

**Primary recommendation:** Use Xilinx XDMA IP configured for AXI4 Memory Mapped interface.
</research_summary>

<standard_stack>
## Standard Stack / IPs

### Core IPs
| Name | Standard | Purpose | Why Standard |
|------|----------|---------|--------------|
| XDMA | PCIe Gen3 | DMA + Bridge | Vendor supported, Drivers available |
| AXI Interconnect | AXI4 | Bus Fabric | Robust, handles CDCs |

### Supporting Tools
| Tool | Feature | Purpose | Use Case |
|------|---------|---------|----------|
| Vivado | ILA | Chipscope Debug | Debugging TLP traffic |
</standard_stack>

<dont_hand_roll>
## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| PCIe TLP Parsing | Custom FSM | Xilinx PHY/Hard Block | Complexity, Compliance |
| DMA Engine | Custom SG List | XDMA IP | Deadlock avoidance, Drivers |
| CDC | Custom Handshake | XPM_CDC | Safety, Constraints included |
</dont_hand_roll>
```
