# Verification Patterns (Hardware)

How to verify that RTL artifacts are substantive, connected, and functionally correct.

<core_principle>
**Existence ≠ Implementation**

A file existing does not mean the feature works. In hardware, "stubs" can synthesize but fail silently (e.g., `assign out = 0`). Verification must check:
1. **Exists** - File is present (`.sv`, `.v`).
2. **Substantive** - Logic is implemented (not just ports and `assign 0`).
3. **Wired** - Instantiated and connected in the hierarchy.
4. **Functional** - Passes simulation and formal checks.
</core_principle>

<stub_detection>

## RTL Stub Patterns

These patterns indicate placeholder code that needs implementation:

**Stub Assignments:**
```bash
# Outputs tied to ground/VDD
grep -E "assign.*=.*'b0;" "$file"
grep -E "assign.*=.*'b1;" "$file"
grep -E "assign.*=.*'h0;" "$file"
```

**Passthrough Stubs:**
```bash
# Input tied directly to output without logic
grep -E "assign.*output_signal.*=.*input_signal;" "$file"
```

**Empty Modules:**
```bash
# Module with no logic body
grep -E "module.*" "$file" && grep -E "endmodule" "$file" # AND check line count < 10
```

**Comment Stubs:**
```bash
grep -E "(TODO|FIXME|XXX|HACK)" "$file"
grep -E "// Logic goes here" "$file"
```

</stub_detection>

<connectivity_checks>

## Connectivity Verification

Unconnected ports or floating nets are fatal in silicon.

**Unconnected Instances:**
```bash
# Check for empty port connections
grep -E "\.\w+\s*\(\s*\)" "$file"
```

**Floating Wires:**
```bash
# Declared but not used (requires linting tool like Verilator)
# verilator --lint-only
```

**Reset/Clock Propagation:**
```bash
# Check if clk/rst are passed to submodules
grep -E "\.clk\s*\(\s*clk\s*\)" "$file"
grep -E "\.rst_n\s*\(\s*rst_n\s*\)" "$file"
```

</connectivity_checks>

<fsm_verification>

## FSM Completeness

Finite State Machines must be safe and complete.

**State Reachability:**
- Are all defined states used in the `case` statement?
- Is there a `default` case assignment?

**Reset State:**
- Does the FSM have an asynchronous/synchronous reset to a known IDLE state?

```systemverilog
// Pattern Check
if (!rst_n)
  state <= IDLE; // REQUIRED
```

</fsm_verification>

<protocol_compliance>

## Protocol Compliance (AXI/APB)

Standard interfaces must obey handshake rules.

**Valid/Ready Handshake:**
- **Deadlock Risk:** `valid` cannot depend on `ready` (combinational loop).
- **Stability:** Once `valid` is asserted, data must remain stable until `ready`.

**Verification Strategy:**
1. **SVA (SystemVerilog Assertions):**
   ```systemverilog
   property p_stable_data;
     @(posedge clk) valid && !ready |=> $stable(data);
   endproperty
   assert property (p_stable_data);
   ```

2. **VIP (Verification IP):**
   - Use AXI VIP in simulation to monitor protocol violations.

</protocol_compliance>

<verification_checklist>

## Quick Verification Checklist

### Module Checklist
- [ ] File exists at expected path (`rtl/core/module.sv`).
- [ ] Compiles without syntax errors (`verilator --lint-only`).
- [ ] No stub assignments (`assign out = 0`).
- [ ] Reset logic present for all sequential logic.
- [ ] Timescale defined (``timescale 1ns/1ps`).

### Connectivity Checklist
- [ ] Instantiated in parent module.
- [ ] All ports connected (no `(.port())` empty connections).
- [ ] Clock and Reset propagated correctly.
- [ ] Parameters passed down correctly.

### Functional Checklist
- [ ] Testbench exists (`tb/tb_module.sv`).
- [ ] Simulation script exists (`Makefile` or `run.sh`).
- [ ] Simulation passes (`TEST PASSED` in log).
- [ ] Waveforms dumped (`dump.vcd`).

</verification_checklist>

<human_verification_triggers>

## When to Require Human Verification

Some things cannot be automated easily:
- **Waveform Inspection:** "Does the arbitration look 'fair'?"
- **CDC Analysis:** "Is this multi-bit signal crossing domains safely?" (needs Spyglass/CDC tool or human review).
- **Architecture Review:** "Is this FIFO depth sufficient for the bandwidth?"
- **Power Analysis:** "Are we clock gating inactive blocks?"

**Request Format:**
```markdown
## Human Verification Required
### 1. Clock Domain Crossing
**File:** rtl/async_fifo.sv
**Check:** Verify gray code pointers are synchronized correctly.
**Action:** Inspect waveforms for `wptr_gray` and `rptr_gray` stability.
```
</human_verification_triggers>
