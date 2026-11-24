# QuillShield Security Skills

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

QuillShield is QuillAudits' AI-Powered Code Review for Solidity. These skills are free and open source. To review a whole Foundry project and check every pull request against that review, use the hosted product: [https://shield.quillaudits.com/](https://shield.quillaudits.com/)

AI agent skills for advanced smart contract security auditing. These skills teach AI agents (Claude, Cursor) the QuillShield methodology for detecting vulnerabilities that traditional static analysis tools miss.

## Quick start

- **Claude:** Install this repo as a Claude plugin; the marketplace is defined in `.claude-plugin/marketplace.json`. Enable the plugins you need for your audit.
- **Cursor:** Reference a skill when auditing — e.g. `@plugins/reentrancy-pattern-analysis/skills/reentrancy-pattern-analysis/SKILL.md` — or copy plugin `skills/` folders into your Cursor skills directory.
- **Use the right skill:** See the table below and the [Skills Overview](#skills-overview) for when to use each plugin.

## Skills Overview

### 1. Behavioral State Analysis (BSA)
**Plugin:** `plugins/behavioral-state-analysis/`

The comprehensive audit methodology. Combines behavioral intent extraction, multi-dimensional threat modeling (economic, access control, state integrity), adversarial simulation with PoC generation, and Bayesian confidence scoring.

**Use when:** Starting a full smart contract security audit, threat modeling DeFi protocols, or generating exploit proof-of-concepts.

### 2. Semantic Guard Analysis
**Plugin:** `plugins/semantic-guard-analysis/`

Detects logic vulnerabilities by finding functions that bypass security checks (require statements, modifiers) that the contract's own code consistently applies elsewhere. Based on the Consistency Principle: "A smart contract is its own specification."

**Use when:** Looking for missing access controls, forgotten pause checks, inconsistent modifiers, or logic bugs invisible to pattern-matching tools.

### 3. State Invariant Detection
**Plugin:** `plugins/state-invariant-detection/`

Automatically infers mathematical relationships between state variables (sum, conservation, ratio, monotonic, synchronization) then finds functions that violate them. Catches the vulnerabilities behind the biggest DeFi hacks.

**Use when:** Auditing for supply/balance mismatches, broken tokenomics, accounting desynchronization, or conservation law violations.

### 4. Reentrancy Pattern Analysis
**Plugin:** `plugins/reentrancy-pattern-analysis/`

Systematically detects all reentrancy variants — classic, cross-function, cross-contract, read-only, and ERC-777/ERC-1155 callback reentrancy. Builds call graphs, verifies CEI pattern compliance, and traces state changes relative to external call positions.

**Use when:** Auditing contracts with external calls, ETH transfers, token interactions, or multi-contract architectures. Covers the most infamous smart contract vulnerability class.

### 5. Oracle & Flash Loan Analysis
**Plugin:** `plugins/oracle-flashloan-analysis/`

Detects price oracle manipulation and flash loan attack vectors — the most common DeFi attack combination. Classifies oracle trust models (Chainlink, TWAP, spot price), identifies stale prices, circular dependencies, and flash loan atomicity exploitation.

**Use when:** Auditing DeFi protocols that depend on price data, oracle integrations, lending protocols, or any contract accessible via flash loans.

### 6. Proxy & Upgrade Safety
**Plugin:** `plugins/proxy-upgrade-safety/`

Detects vulnerabilities in upgradeable proxy architectures — storage layout collisions, uninitialized implementations, function selector clashing, and upgrade path safety. Covers Transparent, UUPS, Beacon, Diamond (EIP-2535), and Minimal proxy patterns.

**Use when:** Auditing upgradeable contracts, reviewing implementation upgrades, or analyzing delegatecall architectures. Critical for the 54.2% of Ethereum contracts that use proxy patterns.

### 7. Input & Arithmetic Safety
**Plugin:** `plugins/input-arithmetic-safety/`

Detects input validation failures (#1 direct exploitation cause at 34.6%) and arithmetic vulnerabilities — precision loss, rounding exploitation, ERC4626 inflation attacks, unsafe casting, and Solidity 0.8+ unchecked block risks.

**Use when:** Auditing fee calculations, share pricing, exchange rates, unchecked blocks, or any public functions with user-supplied parameters.

### 8. External Call Safety
**Plugin:** `plugins/external-call-safety/`

Detects unsafe external call patterns and token integration vulnerabilities. Covers unchecked return values, fee-on-transfer tokens, rebasing tokens, missing ERC20 return values (USDT), callback risks, unsafe approve patterns, and push vs pull payment analysis.

**Use when:** Auditing contracts that interact with external contracts, integrate arbitrary ERC20 tokens, or distribute payments.

### 9. Signature & Replay Analysis
**Plugin:** `plugins/signature-replay-analysis/`

Detects signature replay vulnerabilities affecting 19.63% of signature-using contracts. Covers five replay types (same-chain, cross-chain, cross-contract, nonce-skip, expired), EIP-712 domain verification, ecrecover safety, and permit/permit2 implementation.

**Use when:** Auditing contracts with ecrecover, ECDSA, EIP-712, permit, meta-transactions, or any off-chain signature verification.

### 10. DoS & Griefing Analysis
**Plugin:** `plugins/dos-griefing-analysis/`

Detects Denial of Service and griefing vulnerabilities — unbounded loops, gas limit exhaustion, external call failure DoS, 63/64 gas griefing, storage bloat, timestamp griefing, and self-destruct force-feeding.

**Use when:** Auditing contracts with batch operations, loops over user data, reward distribution, or logic depending on `address(this).balance`.

## Architecture

```
qs_skill/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   ├── behavioral-state-analysis/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── README.md
│   │   └── skills/behavioral-state-analysis/
│   │       ├── SKILL.md
│   │       └── references/
│   │           ├── threat-engines.md
│   │           └── confidence-scoring.md
│   ├── semantic-guard-analysis/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── README.md
│   │   └── skills/semantic-guard-analysis/
│   │       ├── SKILL.md
│   │       └── references/
│   │           ├── detection-algorithm.md
│   │           └── case-studies.md
│   ├── state-invariant-detection/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── README.md
│   │   └── skills/state-invariant-detection/
│   │       ├── SKILL.md
│   │       └── references/
│   │           ├── invariant-types.md
│   │           └── case-studies.md
│   ├── reentrancy-pattern-analysis/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── README.md
│   │   └── skills/reentrancy-pattern-analysis/
│   │       ├── SKILL.md
│   │       └── references/
│   │           ├── reentrancy-variants.md
│   │           └── case-studies.md
│   ├── oracle-flashloan-analysis/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── README.md
│   │   └── skills/oracle-flashloan-analysis/
│   │       ├── SKILL.md
│   │       └── references/
│   │           ├── oracle-types.md
│   │           └── flash-loan-vectors.md
│   ├── proxy-upgrade-safety/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── README.md
│   │   └── skills/proxy-upgrade-safety/
│   │       ├── SKILL.md
│   │       └── references/
│   │           ├── proxy-patterns.md
│   │           └── storage-collision-detection.md
│   ├── input-arithmetic-safety/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── README.md
│   │   └── skills/input-arithmetic-safety/
│   │       ├── SKILL.md
│   │       └── references/
│   │           ├── precision-patterns.md
│   │           └── validation-checklist.md
│   ├── external-call-safety/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── README.md
│   │   └── skills/external-call-safety/
│   │       ├── SKILL.md
│   │       └── references/
│   │           ├── weird-erc20.md
│   │           └── call-safety-patterns.md
│   ├── signature-replay-analysis/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── README.md
│   │   └── skills/signature-replay-analysis/
│   │       ├── SKILL.md
│   │       └── references/
│   │           ├── replay-taxonomy.md
│   │           └── eip712-checklist.md
│   └── dos-griefing-analysis/
│       ├── .claude-plugin/plugin.json
│       ├── README.md
│       └── skills/dos-griefing-analysis/
│           ├── SKILL.md
│           └── references/
│               ├── dos-patterns.md
│               └── gas-griefing-vectors.md
└── README.md
```

## How the Skills Relate

```
                    ┌─────────────────────────────────────────────┐
                    │       Behavioral State Analysis (BSA)        │
                    │       Full Audit Methodology                 │
                    │                                             │
                    │  Phase 1: Behavioral Decomposition           │
                    │  Phase 2: Multi-Dimensional Threat Model     │
                    │  Phase 3: Adversarial Simulation             │
                    │  Phase 4: Confidence Scoring                 │
                    └──────────┬──────────────┬───────────────────┘
                               │              │
             ┌─────────────────┴──────┐ ┌─────┴─────────────────────┐
             │   Original Layers      │ │   Extended Layers          │
             │                        │ │                            │
             │  Layer 1: Semantic     │ │  Layer 3: Reentrancy       │
             │    Guard Analysis      │ │  Layer 4: Oracle/Flash     │
             │  Layer 2: State        │ │  Layer 5: Proxy/Upgrade    │
             │    Invariant Detection │ │  Layer 6: Input/Arithmetic │
             │                        │ │  Layer 7: External Calls   │
             │                        │ │  Layer 8: Signature/Replay │
             │                        │ │  Layer 9: DoS/Griefing     │
             └────────────────────────┘ └────────────────────────────┘
```

## OWASP Smart Contract Top 10 Coverage

| OWASP Category | QuillShield Skill | Coverage |
|----------------|-------------------|----------|
| SC01: Access Control | Semantic Guard Analysis | Full |
| SC02: Oracle Manipulation | Oracle & Flash Loan Analysis | Full |
| SC03: Logic Errors | BSA + State Invariant Detection | Full |
| SC04: Input Validation | Input & Arithmetic Safety | Full |
| SC05: Reentrancy | Reentrancy Pattern Analysis | Full |
| SC06: Unchecked External Calls | External Call Safety | Full |
| SC07: Flash Loan Attacks | Oracle & Flash Loan Analysis | Full |
| SC08: Integer Overflow | Input & Arithmetic Safety | Full |
| SC09: Insecure Randomness | BSA Threat Engines | Partial |
| SC10: DoS Attacks | DoS & Griefing Analysis | Full |

**Beyond OWASP:**

| Category | QuillShield Skill |
|----------|-------------------|
| Proxy/Upgrade Vulnerabilities | Proxy & Upgrade Safety |
| Signature Replay Attacks | Signature & Replay Analysis |
| Token Integration (Weird ERC20) | External Call Safety |
| MEV/Frontrunning | Oracle & Flash Loan Analysis |

## Multi-Layer Severity Matrix

| Layer 1 (Guard) | Layer 2 (Invariant) | Layer 3+ (Extended) | Combined Severity |
|-----------------|---------------------|---------------------|-------------------|
| Missing Guard   | Breaks Invariant    | Additional Vuln     | **CRITICAL**      |
| Missing Guard   | Breaks Invariant    | No Additional       | **CRITICAL**      |
| Missing Guard   | No Break            | Additional Vuln     | **HIGH**          |
| Missing Guard   | No Break            | No Additional       | **HIGH**          |
| No Issue        | Breaks Invariant    | Additional Vuln     | **HIGH**          |
| No Issue        | Breaks Invariant    | No Additional       | **HIGH**          |
| No Issue        | No Break            | Additional Vuln     | **MEDIUM-HIGH**   |
| No Issue        | No Break            | No Additional       | **LOW/INFO**      |

## Source Research

These skills are derived from the QuillShield Semantic State Protocol research and augmented with:

- OWASP Smart Contract Top 10 (2025)
- CertiK Hack3d Web3 Security Report (2024-2025)
- Halborn Top 100 DeFi Hacks Analysis ($10.77B in losses)
- Trail of Bits, OpenZeppelin, and Spearbit audit methodologies
- Academic research on hybrid AI + symbolic execution auditing
- Real-world exploit post-mortems (The DAO, Euler, Mango Markets, Curve, Nomad Bridge, etc.)

## Contributing

We welcome contributions: fixes to existing skills, new references, or new plugins. Please read [CONTRIBUTING.md](CONTRIBUTING.md) for how to propose changes and open pull requests.

## License

This project is licensed under the [MIT License](LICENSE).

## Security

To report a security concern (e.g. in the methodology or documentation), see [SECURITY.md](SECURITY.md).

## Code of Conduct

This project adheres to the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you agree to uphold it.
