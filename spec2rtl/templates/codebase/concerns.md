# Design Constraints & Concerns

## PPA (Power, Performance, Area)
- **Target Frequency:** [e.g., 250 MHz]
- **Setup Slack:** > 0ns (WNS)
- **Hold Slack:** > 0ns (WHS)
- **Area Budget:** < [N]% LUTs, < [M]% BRAMs

## CDC (Clock Domain Crossing)
- **Async Interfaces:** [List domains e.g., PCIe -> Sys]
- **Synchronization:** [e.g., 2-FF synchronizers, Async FIFOs]
- **Risks:** [e.g., Gray code pointer crossings, Handshake stability]

## Critical Warnings
- **Lint:** [e.g., Width mismatches, Combinational loops]
- **Synthesis:** [e.g., Latch inferred, Multi-driven nets]
