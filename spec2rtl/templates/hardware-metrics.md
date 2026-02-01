# Hardware Metrics Report

**Phase:** {PHASE_NAME}
**Date:** {DATE}
**Commit:** {COMMIT_HASH}

## 1. Timing Analysis (Static Timing Analysis)

| Clock Domain | Frequency (MHz) | WNS (ns) | TNS (ns) | Hold WNS (ns) | Status |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `clk_sys` | 200 | +0.45 | 0.00 | +0.02 | PASS |
| `clk_mem` | 400 | -0.12 | -5.4 | +0.01 | **FAIL** |

*   **WNS**: Worst Negative Slack (Must be > 0)
*   **TNS**: Total Negative Slack

## 2. Resource Utilization (Area)

| Resource | Used | Available | Utilization % |
| :--- | :--- | :--- | :--- |
| LUTs | 12,450 | 20,800 | 59.8% |
| Flip-Flops | 8,200 | 41,600 | 19.7% |
| BRAM | 14 | 50 | 28.0% |
| DSP Slices | 4 | 90 | 4.4% |

## 3. Power Estimation

| Domain | Dynamic (mW) | Static (mW) | Total (mW) |
| :--- | :--- | :--- | :--- |
| Core Logic | 145.2 | 22.1 | 167.3 |
| I/O | 34.0 | 5.0 | 39.0 |
| **Total** | **179.2** | **27.1** | **206.3** |

## 4. Verification Coverage

| Metric | Coverage % | Goal % | Status |
| :--- | :--- | :--- | :--- |
| Line Coverage | 98.5% | 100% | ⚠️ |
| Toggle Coverage | 87.0% | 90% | ⚠️ |
| FSM States | 100% | 100% | ✓ |
| Functional Points | 100% | 100% | ✓ |

## 5. Linter Summary

*   **Errors:** 0
*   **Warnings:** 3 (2 Unused signals, 1 Width mismatch)
*   **Waivers:** 1 (CDC synchronizer false path)
