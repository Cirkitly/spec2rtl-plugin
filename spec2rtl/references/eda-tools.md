# EDA Tool Reference

This reference guides the `s2r-eda-orchestrator` and `s2r-executor` on how to invoke common Hardware Design & Verification tools via the command line.

## Simulators

### Verilator (Fast, Cycle-Accurate, Open Source)
```bash
# Linting
verilator --lint-only -Wall top.v

# Building & Running Simulation
verilator --binary -j 0 -Wall --trace --top-module top_module sim_main.cpp top.v
./obj_dir/Vtop_module
```

### Icarus Verilog (Event-Driven, Open Source)
```bash
# Compile
iverilog -g2012 -o sim_out -s top_tb top.v tb.v

# Run
vvp sim_out
```

### Synopsys VCS
```bash
# Compile & Run
vcs -sverilog -debug_access+all -kdb -l compile.log top.v tb.v
./simv -l run.log +UVM_TESTNAME=my_test
```

### Cadence Xcelium
```bash
xrun -sv -64bit -access +rwc top.v tb.v
```

## Linting & Formatting

### Verible
```bash
# Lint
verible-verilog-lint --rules_config .rules.verible **/*.v

# Format
verible-verilog-format --inplace **/*.v
```

## Synthesis (FPGA/ASIC)

### Xilinx Vivado (FPGA)
```bash
# Batch Mode Synthesis
vivado -mode batch -source scripts/synth.tcl -tclargs -part xc7a35tcpg236-1
```
*synth.tcl snippet:*
```tcl
read_verilog [glob src/*.v]
synth_design -top top_module -part $part
report_utilization -file util.rpt
report_timing_summary -file timing.rpt
```

### Yosys (Open Source Synthesis)
```bash
# Synthesize to JSON or BLIF
yosys -p "synth_ice40 -top top_module -json top.json" src/*.v
```

## Waveform Viewers

### GTKWave
```bash
gtkwave dump.vcd
```
