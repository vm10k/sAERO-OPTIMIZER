# sAERO Optimizer

A client-side yield optimization dashboard and decision engine for Aerodrome MetaDEX on Base. The application calculates real, dilution-adjusted returns across continuous 48-hour settlement cycles and computes optimal voting power distribution using fractional marginal yield analysis.

---

## Network Support Notice

**Important:** This application currently operates exclusively on the **Base Mainnet (Chain ID: 8453)**. 

While the interface contains references to other Superchain networks for future expansion, contract queries, factory crawlers, and voting calculations are currently restricted to Base. Do not attempt to route or read non-Base positions with this release.

---

## Overview

Classical ve-token governance models rely on fixed 7-day epochs. Aerodrome MetaDEX introduces continuous rolling 48-hour settlement windows governed on-chain by the `LeafVoter` system. 

Standard yield dashboards display static or historical Stake APR. For positions with significant voting power, casting votes into smaller pools immediately dilutes the pool's reward share, causing realized yield to drop drastically below the advertised APR. 

sAERO Optimizer solves this by:
1. Fetching live on-chain gauge states, pending fees, reserves, and incentives directly via Multicall3.
2. Simulating voter dilution before votes are cast.
3. Slicing user voting power into 100 discrete micro-increments and routing each increment to the gauge producing the highest marginal dollar gain.
4. Projecting full expected returns in both USD ($) and native AERO tokens across 48-hour and annualized periods.

---

## Features

- **Direct On-Chain Aggregation:** Reads pool factories, gauges, claimable fees, and active voter stakes via Multicall3 without relying on proprietary backend indexers.
- **Dynamic Dilution Modeling:** Recalculates projected APR based on how your specific veAERO power will dilute target pools.
- **Marginal Optimization Engine:** Allocates voting power in 1% steps to maximize absolute dollar return across qualified high-liquidity gauges.
- **Full Outcome Reporting:** Displays total cycle return (48 hours) and annualized return broken down into full numerical representations of USD value and AERO token quantities.
- **Local veNFT Persistence:** veNFT token IDs are stored in browser `localStorage` and automatically loaded on refresh, eliminating redundant inputs while preventing UI render locks during initial scanning.
- **Zero-Backend Architecture:** Runs entirely in the browser using static HTML, JavaScript (ES6+), and Ethers.js v6.

---

## MetaDEX Base Contract Architecture

The application interfaces directly with the following canonical contracts on Base Mainnet:

| Contract Name | Address | Function |
| :--- | :--- | :--- |
| **LeafVoter** | `0x382409fad8503FbCdbB27A4211551fa03e884078` | Gauge state accounting and 48-hour settlement |
| **VotingEscrow** | `0xBF7977E1521e9a3032bf7243Ee4712f311f13915` | veNFT balance checks, lock durations, total voting power |
| **Factory Registry** | `0x7b2A2c12dC5Ea2a31455952a05031C3E4BB28F3E` | Discovery and mapping for pool factories and gauges |
| **Multicall3** | `0xcA11bde05977b3631167028862bE2a173976CA11` | Batched RPC reads for pool reserves and gauge reward rates |
| **AERO Token** | `0x940181a94A35A4569E4529A3CDfB74e38FD98631` | Protocol token valuation and price reference |

---

## How the Marginal Optimizer Works

1. **Filtering:** Filters candidate pools based on liquidity thresholds (minimum $5,000 48-hour volume or $25,000 TVL) to prevent capital trapping in illiquid gauges.
2. **Chunking:** Breaks the total user voting power ($V$) into $N = 100$ uniform chunks ($v_c = \frac{V}{100}$).
3. **Marginal Analysis:** For each chunk $k \in [1, 100]$:
   - Evaluates each qualified pool $i$.
   - Computes the user's current reward share:
     $$S_{current} = \frac{v_{allocated, i}}{V_{pool, i} + v_{allocated, i}}$$
   - Computes the next share if chunk $v_c$ is added:
     $$S_{next} = \frac{v_{allocated, i} + v_c}{V_{pool, i} + v_{allocated, i} + v_c}$$
   - Calculates marginal dollar gain:
     $$\Delta R = R_{48h, i} \times (S_{next} - S_{current})$$
   - Assigns the chunk to the pool with the highest marginal dollar gain ($\Delta R$).
4. **Resolution:** Outputs a multi-gauge distribution that maximizes realized yield for the exact token balance provided.

---

## Getting Started

### Prerequisites

- A modern web browser (Chrome, Brave, Firefox, Edge).
- An active Base RPC endpoint (default public endpoints are configured via FallbackProvider).

### Running Locally

1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/saero-optimizer.git
   cd saero-optimizer
   ```

2. Serve the directory using any static file server:

   Using Python 3:
   ```bash
   python3 -m http.server 8080
   ```

   Using Node.js (via `serve`):
   ```bash
   npx serve .
   ```

3. Open your browser and navigate to:
   ```
   http://localhost:8080
   ```

---

## Configuration and Usage

1. **Load veNFT:**
   - Enter one or more comma-separated veNFT token IDs into the quick input bar on the Pools view or under the Settings tab.
   - Click "Confirm & Save Settings". The application saves IDs to `localStorage` and queries `balanceOfNFT()` from the `VotingEscrow` contract.
2. **Inspect Projected APR:**
   - The pool list automatically updates every gauge's "Projected APR" column, reflecting the expected yield should you allocate 100% of your power to that specific pool.
3. **Run Marginal Optimization:**
   - Click "Optimize For My Power".
   - Open "My Votes" via the sidebar to view your optimal distribution, 48-hour dollar/token return, annualized dollar/token yield, and pool-by-pool breakdown.
4. **Manual Adjustment:**
   - Sliders and numeric percentage inputs allow you to fine-tune allocation percentages manually while observing changes in projected revenue in real time.

---

## License

MIT License. See `LICENSE` for further details.
