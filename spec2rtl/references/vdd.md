<overview>
Verification Driven Development (VDD) is the hardware equivalent of TDD. It prioritizes defining "Success" (Verification) before "Implementation" (RTL).
In hardware, bugs found at integration are 10x more expensive than block-level, and silicon bugs are 1000x more expensive. VDD shifts verification left.

**Principle:** If you can't assert it, you can't design it. Write the SVA (Assertion) or Test Case before writing the RTL.

**Key insight:** VDD reduces "Verification Debt". Instead of accumulating a mountain of unverified RTL that requires weeks of debug at the end, every module is proven as it is written.
</overview>

<when_to_use_vdd>
## When VDD Improves Quality

**VDD candidates (create a VDD plan):**
- Control Logic & FSMs (State transitions, deadlocks)
- Standard Interfaces (AXI, APB, SPI compliance)
- Arbiters and FIFOs (Fairness, Overflow/Underflow protection)
- Data path arithmetic (ALU correctness)
- Protocol Converters

**Skip VDD (use standard implementation):**
- Top-level wiring (Integration only)
- Pad rings / IO instantiation
- Simple structural glue logic
- Vendor IP instantiation (Assume IP is correct, verify integration)

**Heuristic:** Can you write `assert property (@(posedge clk) req |-> ##[1:3] gnt)` before writing the RTL?
→ Yes: Use VDD.
→ No: Use standard plan.
</when_to_use_vdd>

<execution_flow>
## Assert-Implement-Verify Cycle

**1. ASSERT - Write failing checks:**
1. Create the module interface (ports only).
2. Create the Bind file (`module_sva.sv`) or Testbench (`tb_module.sv`).
3. Write **SystemVerilog Assertions (SVA)** for the requirements.
   - `a_req_gnt: assert property (@(posedge clk) req |-> ##[1:5] gnt);`
   - `a_no_overflow: assert property (@(posedge clk) full |-> !push);`
4. Run simulation/formal. It **MUST FAIL** (or compile error due to missing signals).
5. Commit: `test(01-01): add assertions for arbiter`

**2. IMPLEMENT - Implement to pass:**
1. Write the RTL body (`module.sv`).
2. Implement the FSM/Logic to satisfy the assertions.
3. Run simulation/formal. It **MUST PASS**.
4. Commit: `feat(01-01): implement arbiter logic`

**3. VERIFY/REFACTOR (Optional):**
1. Optimize for Area/Timing if needed.
2. Clean up code structure.
3. Run regression - **MUST STILL PASS**.
4. Commit: `refactor(01-01): optimize arbiter muxing`

**Result:** High-confidence RTL with built-in coverage and safety checks.
</execution_flow>

<sva_guide>
## SVA Cheat Sheet for VDD

**Immediate Assertions (Combinational):**
```systemverilog
// Check inside always_comb
always_comb begin
  if (valid) assert (data !== 'x) else $error("X on valid data");
end
```

**Concurrent Assertions (Temporal):**
```systemverilog
// Request must be followed by Grant within 5 cycles
property p_req_gnt;
  @(posedge clk) req |-> ##[1:5] gnt;
endproperty
a_req_gnt: assert property (p_req_gnt);

// Mutex: Only one grant at a time
a_mutex: assert property (@(posedge clk) $onehot0(gnt_bus));
```

**Bind Files (Separating Verification from Design):**
Use `bind` to attach assertions without modifying RTL files.
```systemverilog
// tb_top.sv
bind my_module my_module_sva inst_sva (.*);
```
</sva_guide>

<context_budget>
## Context Budget

VDD plans require higher context awareness (~60%) because the agent must understand both the Design Intent (RTL) and the Verification Intent (TB/SVA) simultaneously.

**Efficiency Tip:** Keep assertions simple and localized. Complex end-to-end checks belong in the Verification Phase, not the Design VDD cycle.
</context_budget>
