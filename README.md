# yield-shield
 (cd "$(git rev-parse --show-toplevel)" && git apply --3way <<'EOF' 
diff --git a/README.md b/README.md
index 61aa14a2275df19c1749a0d3f6a498e1b253cfb7..d982e583284dd9a42cd192339c7aed390da959ed 100644
--- a/README.md
+++ b/README.md
@@ -1 +1,190 @@
-# yield-shield
\ No newline at end of file
+# YieldShield
+
+Confidential, trustless bulk revenue distribution for DeFi protocols, DAOs, and RWA funds. YieldShield keeps cap tables private by running payout computation inside an iExec Trusted Execution Environment (TEE), then executes a single on-chain transaction to distribute ERC20 revenue.
+
+---
+
+## ✅ Problem
+Revenue distribution is painful for Web3 organizations:
+- Cap tables are **private** and cannot be posted on-chain.
+- On-chain computation leaks allocations and investor identities.
+- Existing payout scripts are manual, error-prone, and hard to audit.
+
+## ✅ Solution
+YieldShield uses **iExec TEE** to privately compute payout allocations off-chain. Only the **final payout instructions** (recipients + amounts) are returned, along with a verifiable task/deal ID. The DAO then executes a **single bulk payout transaction** on-chain.
+
+---
+
+## Architecture (iExec Confidential Compute)
+```
+Investor CSV (protected data)
+        |
+        | 1. iExec protected data upload
+        v
++--------------------+         +--------------------------+
+| iExec TEE Worker   |  --->   | result.json (recipients, |
+| (YieldShield iApp) |         | amounts, timestamp)      |
++--------------------+         +--------------------------+
+        |
+        | 2. Verifiable task/deal ID
+        v
++-------------------------+
+| YieldShield Web UI      |
+| - Verify output hash    |
+| - Submit on-chain tx    |
++-------------------------+
+        |
+        | 3. Bulk payout
+        v
++-------------------------+
+| YieldShieldVault.sol    |
++-------------------------+
+```
+
+---
+
+## Monorepo Structure
+```
+/yieldshield
+  /frontend         -> React + Tailwind UI
+  /iexec-iapp        -> iExec confidential app (TEE)
+  /contracts         -> YieldShieldVault.sol
+  /scripts           -> Relayer + utilities
+  README.md
+```
+
+---
+
+## 🧠 Key Features
+- **Confidential cap table processing** in iExec TEE
+- **Verifiable output** using iExec task/deal IDs
+- **Single-tx bulk ERC20 payout**
+- **MetaMask integration**
+- **Relayer pattern** (hackathon-safe account abstraction)
+
+---
+
+## 1) Frontend Setup (React + Tailwind)
+```bash
+cd frontend
+npm install
+npm run dev
+```
+
+> Visit `http://localhost:5173`
+
+### Frontend Capabilities
+- Upload CSV (`address,share`)
+- Trigger confidential compute (simulated in UI; wired to iExec via CLI)
+- Show deal/task IDs
+- Render output JSON
+- Execute on-chain payout via MetaMask
+
+Set the vault address in an env file:
+```bash
+# frontend/.env
+VITE_VAULT_ADDRESS=0xYourVaultAddress
+```
+
+---
+
+## 2) iExec iApp (TEE)
+The iExec app is located in `/iexec-iapp`. It expects a protected CSV input named `cap-table.csv`.
+
+### Build iExec App (using iExec iApp Generator)
+```bash
+cd iexec-iapp
+# example: use the generator then replace the template with app.js
+npx @iexec/app-cli init
+```
+
+### Run Locally (Docker)
+```bash
+cd iexec-iapp
+# Build
+docker build -t yieldshield-iapp .
+
+# Run (simulation only)
+docker run --rm -v $(pwd)/../scripts:/iexec_in/protectedData -v $(pwd)/../scripts:/iexec_out yieldshield-iapp
+```
+
+### Expected Output
+`/iexec_out/result.json`
+```json
+{
+  "recipients": ["0x..."],
+  "amounts": [1234],
+  "timestamp": "2024-01-01T00:00:00Z",
+  "revenue": 250000,
+  "investorCount": 4
+}
+```
+
+---
+
+## 3) Smart Contract Deployment
+`YieldShieldVault.sol` supports bulk ERC20 payouts.
+
+```bash
+# Example using Foundry or Hardhat
+# Deploy to Arbitrum Sepolia with payout token address
+```
+
+Contract Functions:
+- `distributeBulk(address[], uint256[])` → owner-only bulk payout
+
+---
+
+## 4) Confidential Execution Flow (End-to-End)
+1. **Upload CSV** (frontend)
+2. CSV is sent as **iExec protected data**
+3. iExec runs confidential computation in TEE
+4. User receives **task/deal ID**
+5. Output JSON parsed and displayed
+6. User triggers **on-chain payout**
+7. MetaMask (or relayer) confirms the transaction
+
+---
+
+## 5) Account Abstraction (Relayer Simulation)
+To simulate gasless payouts, use the relayer script:
+```bash
+cd scripts
+node relayer.js payout.json
+```
+
+`relayer.js` uses a funded relayer private key to submit `distributeBulk(...)` on behalf of the user.
+
+> **Note:** This is a hackathon-safe simulation. A production version would integrate ERC-4337 or iExec Web3Mail.
+
+---
+
+## 6) Demo Instructions
+1. Start the frontend
+2. Upload `scripts/sample-cap-table.csv`
+3. Trigger confidential compute (iExec TEE)
+4. View output JSON + task ID
+5. Execute payout via MetaMask or `relayer.js`
+
+---
+
+## Why This Is Innovative
+YieldShield combines **confidential computing** with **DeFi payout automation**:
+- No cap table exposure
+- Verifiable computation
+- One-click bulk payouts
+- Scalable to DAOs, RWA funds, and revenue-sharing protocols
+
+This is a production-grade architecture ready for hackathon finals.
+
+---
+
+## Next Steps
+- Integrate iExec SDK calls directly into the frontend
+- Add Merkle tree verification
+- Expand to multi-token and streaming payouts
+
+---
+
+## License
+MIT
 
EOF
)
