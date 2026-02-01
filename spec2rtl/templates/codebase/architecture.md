# RTL Hierarchy & Architecture

## Top Level
- **Module:** `top.sv`
- **Interfaces:** [e.g., Clock, Reset, UART, DDR, PCIe]

## Bus Fabric
- **Protocol:** [e.g., AXI4, AXI-Lite, APB, TileLink]
- **Interconnect:** [e.g., SmartConnect, Crossbar]
- **Addressing:** [e.g., 32-bit, 64-bit]

## Clock Domains
- **clk_sys:** [Frequency, e.g. 100MHz] - Main logic
- **clk_ddr:** [Frequency] - Memory interface
- **clk_pcie:** [Frequency] - PCIe core

## Reset Tree
- **rst_n:** Async Active Low (Global)
- **soft_rst:** Sync Active High (Register controlled)
