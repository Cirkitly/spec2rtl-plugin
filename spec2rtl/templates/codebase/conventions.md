# RTL Coding Conventions

## Naming
- **Inputs:** `*_i` (e.g., `clk_i`, `data_i`)
- **Outputs:** `*_o` (e.g., `valid_o`)
- **Active Low:** `*_n` (e.g., `rst_n`)
- **Types:** `_t` (typedef), `_e` (enum), `_pkg` (package)

## Style
- **FSM:** 3-process (Next State, Output, Sequential) or 2-process.
- **Sequential:** `always_ff @(posedge clk)`
- **Combinational:** `always_comb`
- **Reset:** Asynchronous Active Low (unless target requires otherwise).

## Linting rules
- No implicit nets (`default_nettype none`).
- No latches (`if` without `else` in comb).
- Width matching required.
