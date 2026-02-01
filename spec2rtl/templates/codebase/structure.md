# Directory Structure

## Layout
- `rtl/` - Design Sources (`.sv`, `.v`, `.vhd`)
  - `core/` - Core logic
  - `ips/` - IP wrappers
- `tb/` - Simulation Sources
  - `uvm/` - UVM agents/env
  - `tests/` - Specific test scenarios
- `sim/` - Simulation work directory
- `syn/` - Synthesis/Implementation scripts and results
- `constraints/` - Timing (`.xdc`, `.sdc`) and Pinout
- `scripts/` - TCL, Python, Makefiles
- `docs/` - Specifications and Registers

## File Lists
- `rtl/manifest.f` - List of design files
- `tb/tb_manifest.f` - List of test files
