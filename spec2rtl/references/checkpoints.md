<overview>
Plans execute autonomously. Checkpoints formalize the interaction points where human verification or physical intervention is needed.

**Core principle:** Claude automates the EDA tools (Simulation, Synthesis, Place & Route). Human handles the **Physical World** (Cables, Boards, Scopes) and **Visual Judgment** (Waveforms, Floorplans).

**Golden rules:**
1.  **If a tool can run via CLI, Claude runs it** - Synthesis, Simulation, Bitstream Generation.
2.  **Claude sets up the environment** - Launch OpenOCD, start Vivado Hardware Manager.
3.  **User handles physics** - Plugging in cables, cycling power, pressing buttons.
4.  **User handles aesthetics/judgment** - "Does this state machine look efficient?", "Is this CDC safe?"
</overview>

<checkpoint_types>

<type name="human-verify">
## checkpoint:human-verify (Visual/Physical Checks)

**When:** Claude completed tool execution, human must verify output quality or physical behavior.

**Use for:**
- **Waveform Inspection:** Glitch detection, X-propagation checks.
- **Physical Indicators:** LEDs blinking, buzzer sounding, screen output.
- **Lab Measurements:** Current draw, voltage rails, logic analyzer captures.
- **Floorplan Review:** Congestion hotspots, clock tree topology.

**Structure:**
```xml
<task type="checkpoint:human-verify" gate="blocking">
  <what-built>[RTL Module / Bitstream / Simulation]</what-built>
  <how-to-verify>
    [Exact steps - "Look at LED D1", "Open dump.vcd"]
  </how-to-verify>
  <resume-signal>Type "approved" or describe issues</resume-signal>
</task>
```

**Example: Simulation Waveform**
```xml
<task type="auto">
  <name>Run UART Simulation</name>
  <action>Run `make sim` to generate `dump.vcd`</action>
  <verify>Log shows "TEST PASSED"</verify>
</task>

<task type="checkpoint:human-verify">
  <what-built>UART Receiver RTL + Waveform dump.vcd</what-built>
  <how-to-verify>
    Open `dump.vcd` in GTKWave.
    Verify `rx_valid` goes high exactly 1 cycle after stop bit.
    Check that `rx_data` is stable when `rx_valid` is high.
  </how-to-verify>
</task>
```

**Example: Board Test**
```xml
<task type="auto">
  <name>Program FPGA</name>
  <action>Run `openocd -f board/digilent_arty.cfg -c "program top.bit verify exit"`</action>
  <verify>Exit code 0</verify>
</task>

<task type="checkpoint:human-verify">
  <what-built>Bitstream loaded to Arty A7</what-built>
  <how-to-verify>
    1. Press Button 0.
    2. Verify LED 0 toggles.
    3. Verify UART console prints "Button Pressed".
  </how-to-verify>
</task>
```
</type>

<type name="human-action">
## checkpoint:human-action (Physical Setup)

**When:** The automation is blocked by the laws of physics.

**Use for:**
- Connecting JTAG/USB cables.
- Power cycling the board.
- Changing jumper settings (Boot mode).
- Connecting external probes (Oscilloscope).

**Structure:**
```xml
<task type="checkpoint:human-action" gate="blocking">
  <action>[Physical action required]</action>
  <instructions>
    [Specific steps]
  </instructions>
  <verification>[How Claude will check you did it]</verification>
</task>
```

**Example: JTAG Connection**
```xml
<task type="checkpoint:human-action">
  <action>Connect JTAG Adapter</action>
  <instructions>
    Connect the Digilent HS2 JTAG cable to Header J12.
    Ensure the board is powered on (Red LED lit).
  </instructions>
  <verification>I will run `lsusb` to confirm device visibility.</verification>
</task>
```
</type>

<type name="decision">
## checkpoint:decision (Architecture/IP)

**When:** A trade-off must be made (Area vs Speed, Buy vs Build).

**Use for:**
- **IP Selection:** Use Xilinx IP vs Open Source RTL.
- **Pinout Definition:** Which physical pins to assign.
- **Protocol Choice:** AXI4 vs AXI-Lite vs APB.

**Example:**
```xml
<task type="checkpoint:decision">
  <decision>Select FIFO Implementation</decision>
  <options>
    <option id="xilinx_ip">
      <name>Xilinx FIFO Generator</name>
      <pros>Guaranteed timing, easy to resize</pros>
      <cons>Vendor lock-in, hard to simulate in Verilator</cons>
    </option>
    <option id="rtl_custom">
      <name>Custom SystemVerilog FIFO</name>
      <pros>Portable, transparent, Verilator-friendly</pros>
      <cons>Need to verify CDC logic manually</cons>
    </option>
  </options>
</task>
```
</type>

</checkpoint_types>

<execution_protocol>
**Claude's Role:**
1.  Prepare all files (bitstreams, scripts).
2.  Start any required servers (OpenOCD, Vivado hw_server).
3.  Present the checkpoint.
4.  Wait for user confirmation.
5.  Verify result (grep logs, check USB devices) before proceeding.

**User's Role:**
1.  Perform the physical/visual action.
2.  Type "done" or "approved".
</execution_protocol>
