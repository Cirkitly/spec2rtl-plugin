# Verification Methodology

## Strategy
- **Unit Level:** Self-checking testbenches for leaf modules.
- **Integration:** Connectivity and basic transaction tests.
- **System:** Firmware-driven simulation or full UVM testbench.

## Tools
- **Simulator:** [Verilator / XSim / VCS]
- **Framework:** [UVM / Cocotb / SV-Unit]
- **Waves:** [FST / VCD / WDB]

## Coverage Goals
- **Line/Toggle:** > 90%
- **Functional:** 100% of defined covergroups
- **Asserts:** 0 Failures

## Regression
- **Command:** `make test` or `python run.py`
- **CI:** [GitHub Actions / Jenkins flows]
