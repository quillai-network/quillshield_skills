---
name: semantic-guard-analysis
description: Detects logic vulnerabilities in smart contracts by analyzing guard-state consistency patterns. Identifies functions that bypass security checks (require, modifiers) that other functions consistently apply. Uses the Consistency Principle — a contract is its own specification. Use when auditing smart contracts for missing access controls, inconsistent pause checks, logic bugs, forgotten modifiers, or when traditional tools report no issues but logic errors may exist.
---

# Semantic Guard Analysis

Detect logic vulnerabilities by finding functions that **violate the contract's own internal guard patterns**. Unlike pattern-matching tools, this approach uses the contract's consistent behavior as its specification.

## When to Use

- Auditing smart contracts where traditional tools find nothing suspicious
- Looking for missing `require` checks, forgotten modifiers, inconsistent access control
- Analyzing contracts with emergency/admin functions that might bypass safety mechanisms
- Detecting logic bugs that are syntactically correct but semantically dangerous
- When you suspect "forgotten check" vulnerabilities

## When NOT to Use

- Pure state-state invariant analysis (use state-invariant-detection)
- Full multi-dimensional audit (use behavioral-state-analysis)
- Code quality or gas optimization reviews

## Core Principle: The Consistency Hypothesis

> **"A smart contract is its own specification."**

Instead of checking against external rules, analyze what the contract **claims to enforce**, then find where it **breaks its own rules**.

> If a critical state variable (like user balances) is protected by a security check (like a pause mechanism) in 90% of functions, the 10% without that check are likely vulnerabilities.

## The Three-Phase Detection Architecture

### Phase 1: AST Extraction & State Mapping

Parse the Solidity code and build a **State Interaction Matrix**.

**For each state variable, track every function that touches it:**

```
State Variable: balance
├─ deposit()        → [WRITE] + Guards: [paused, initialized]
├─ withdraw()       → [WRITE] + Guards: [paused, initialized]
├─ transfer()       → [WRITE] + Guards: [paused]
└─ emergencyWithdraw() → [WRITE] + Guards: [] ⚠️
```

**For each function-variable interaction, record:**

| Attribute | Description |
|-----------|-------------|
| Write Access | Does the function modify this variable? |
| Guard Access | Does the function check this variable in `require()` or `if()`? |
| Read Access | Does the function only read this variable? |

**Extract guard sources:**
- Modifier chains (`onlyOwner`, `nonReentrant`, `whenNotPaused`)
- Explicit `require` statements
- Conditional branches gating state changes
- External calls affecting state
- Event emissions signaling state changes

### Phase 2: Dependency Graph Construction

Build a mathematical model of how variables protect each other.

**Guard Relationship:** If Variable A is checked before Variable B is modified:

```
A → B (A guards B)
```

**Example:**

```
paused ──────┐
             ├──→ balance
initialized ─┘

owner ───→ paused
owner ───→ totalSupply
```

**Frequency Weighting:** Each guard relationship gets a confidence score:

```
Confidence(guard → state) = |functions applying guard| / |functions modifying state|
```

- `paused` guards `balance` in 9/10 functions → 90% confidence
- `owner` guards `totalSupply` in 3/10 functions → 30% confidence (weak)

**Composite Dependencies:** Track multi-variable guards:

```
(owner AND timeLock) → criticalFunction
(paused OR emergency) → userAccess
```

### Phase 3: Anomaly Detection (The Solver)

Identify functions that violate established patterns.

**Algorithm:**

```
For each state variable S that can be modified:
  1. M = all functions that write to S
  2. G = common guards across those functions (above threshold)
  3. V = M \ G (functions that modify without guards)
  4. V is the vulnerability set
```

**Threshold-Based Inference:**

| Guard Frequency | Classification | Action |
|-----------------|---------------|--------|
| ≥ 80% | Strong Invariant | Flag violations as HIGH/CRITICAL |
| 50-79% | Weak Invariant | Flag violations as MEDIUM |
| < 50% | No Pattern | Ignore (too inconsistent) |

**Severity Classification:**

| Bypass Type | Severity |
|-------------|----------|
| Strong invariant on financial state (`balance`, `totalSupply`) | **Critical** |
| Strong invariant on access control (`owner`, admin roles) | **High** |
| Weak invariant on any state | **Medium** |
| Inconsistent pattern with no security implications | **Low/Info** |

**Context-Aware Filtering:**
- Constructor and `initialize()` functions may legitimately bypass patterns
- `view`/`pure` functions cannot modify state — skip
- Proxy pattern `delegatecall` requires special handling
- Emergency functions may intentionally bypass some guards

## Workflow

```
Task Progress:
- [ ] Step 1: Parse contract AST and build State Interaction Matrix
- [ ] Step 2: Identify all state variables and their modifying functions
- [ ] Step 3: Map guards (requires, modifiers) for each function-state pair
- [ ] Step 4: Build dependency graph with frequency weighting
- [ ] Step 5: Run anomaly detection (identify V = M \ G)
- [ ] Step 6: Apply privilege overlay (filter legitimate bypasses)
- [ ] Step 7: Score and report findings
```

## Privilege Overlay System

Not all "bypasses" are vulnerabilities. Apply role-based filtering:

**Role Classification:**

| Role Level | Scrutiny | Rationale |
|------------|----------|-----------|
| Public functions | Highest | Must follow all established patterns |
| Owner/Admin functions | Medium | May bypass operational guards, must be consistent with each other |
| Emergency functions | Lower | Designed for exceptional cases |
| Internal functions | Context-dependent | Analyze based on callers |

**Filtering Rule:**

```
For each function f in vulnerability set V:
  1. Identify function privileges (modifiers, access controls)
  2. Compare with other functions at the SAME privilege level
  3. Flag only if bypass is inconsistent WITHIN privilege tier
```

## Output Format

```markdown
## Guard-State Anomaly Report

### Finding: [Title]

**Function:** `functionName()` at `Contract.sol:L145`
**Severity:** [CRITICAL | HIGH | MEDIUM | LOW]
**Confidence:** [Percentage]

**Issue:** Modifies `[state variable]` without checking `[guard]`

**Pattern Evidence:**
- `function1()` checks `[guard]` before modifying `[state]` ✓
- `function2()` checks `[guard]` before modifying `[state]` ✓
- `functionName()` does NOT check `[guard]` before modifying `[state]` ✗

**Guard Frequency:** X out of Y functions (Z%)

**Security Impact:**
[Explanation of what an attacker can do by exploiting this inconsistency]

**Attack Scenario:**
1. [Step-by-step exploit]

**Recommendation:**
Add `require([guard], "[message]")` before modifying `[state]`,
or document why this function intentionally bypasses the check.
```

## Case Study: The "Forgotten Check"

```solidity
contract Vault {
    mapping(address => uint256) public balance;
    bool public paused;

    function deposit() public payable {
        require(!paused, "Contract paused");       // ✓ checks paused
        balance[msg.sender] += msg.value;
    }

    function withdraw(uint256 amount) public {
        require(!paused, "Contract paused");       // ✓ checks paused
        balance[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }

    function adminWithdraw(address user) public onlyOwner {
        // ✗ Missing paused check!
        uint256 amount = balance[user];
        balance[user] = 0;
        payable(owner).transfer(amount);
    }
}
```

**Detection:**

```
M_balance = {deposit, withdraw, adminWithdraw}
G_paused = {deposit, withdraw}
V = {adminWithdraw}

Result: adminWithdraw() modifies balance without checking paused
Confidence: 66.7% (2/3 functions check paused)
Severity: HIGH (financial state + admin bypass of safety mechanism)
```

For more case studies, see [{baseDir}/references/case-studies.md]({baseDir}/references/case-studies.md).
For the full detection algorithm, see [{baseDir}/references/detection-algorithm.md]({baseDir}/references/detection-algorithm.md).

## Rationalizations to Reject

- "The admin is trusted, so skipping the check is fine" → Compromised admin + missing pause check = unstoppable drain
- "This function is only called internally" → Verify all callers; internal doesn't mean safe
- "The pattern only appears in 2 functions" → Even 2/3 consistency is a signal worth investigating
- "It's an emergency function" → Emergency functions should be MORE carefully guarded, not less
- "Traditional tools said it's fine" → Traditional tools check syntax, not semantic consistency

## Extended Pattern Library: Business Logic Calculation Errors

Beyond guard-state inconsistencies, flag these recurring calculation bugs that cause silent financial harm even when no guards are missing:

### 1. Vesting Cliff Elapsed Miscalculation (`cliff_elapsed_miscalculation`)

**Function to flag:** the `vested()` view function (or equivalent calculation helper).

**Pattern:** After the cliff guard passes (`block.timestamp >= start + cliff`), the contract computes `elapsed = block.timestamp - start` instead of `elapsed = block.timestamp - (start + cliff)`. This includes the cliff period in the linear ramp, so the beneficiary can immediately claim a large lump-sum at cliff expiry rather than starting from zero.

```solidity
// WRONG – includes cliff period in elapsed
if (block.timestamp < g.start + g.cliff) return 0;
uint256 elapsed = block.timestamp - g.start;          // ← bug: should subtract cliff too
return g.total * elapsed / g.duration;

// CORRECT
uint256 elapsed = block.timestamp - (g.start + g.cliff);
uint256 vestingDuration = g.duration - g.cliff;
return g.total * elapsed / vestingDuration;
```

**Vulnerability type:** `cliff_elapsed_miscalculation`

### 2. Flash-Loan Repayment Check Off-By-Amount (`incorrect_repayment_check`)

**Function to flag:** the flash-loan dispatcher (e.g. `flashLoan()`).

**Pattern:** The post-callback balance check reads `balanceAfter >= balanceBefore - amount + fee` instead of `balanceBefore + fee`. Because `balanceBefore` was measured *before* the loan was sent, the correct invariant is that the pool ends up *richer* by `fee`. The incorrect form allows the borrower to keep nearly the entire loan.

**Vulnerability type:** `incorrect_repayment_check`

### 3. Share Inflation / First-Depositor Attack (`share_price_manipulation`)

**Function to flag:** `deposit()` (or `mint()`).

**Pattern:** When `totalShares == 0`, shares are minted 1:1 with assets. An attacker deposits 1 wei, then donates tokens directly to the vault, inflating `totalAssets`. Subsequent depositors receive 0 shares because `assets * 1 / largeBalance` rounds to zero.

**Vulnerability type:** `share_price_manipulation`

### 4. Inverted Deadline Check (`ineffective_deadline`)

**Function to flag:** the swap or action function that uses `>=` instead of `<=` (or vice-versa) for a timestamp deadline.

**Pattern:** `require(block.timestamp >= deadline)` passes only *after* the deadline, making the deadline an activation time rather than an expiry. Contrast with the sibling function that correctly uses `require(deadline >= block.timestamp)`.

**Vulnerability type:** `ineffective_deadline`

### 5. Missing Snapshot in Voting (`missing_snapshot_check`)

**Function to flag:** `castVote()` (or equivalent).

**Pattern:** The proposal records a `snapshotBlock` but `castVote` reads `token.balanceOf(msg.sender)` (live balance) instead of `token.getPastVotes(msg.sender, snapshotBlock)`. Attackers can borrow tokens, vote, then return them — a flash-loan governance attack.

**Vulnerability type:** `missing_snapshot_check`

### 6. Staking Without Reward Debt Initialization (`incorrect_reward_accounting`)

**Function to flag:** `stake()` (or `deposit()` in a rewards vault).

**Pattern:** A user calls `stake()` after `accRewardPerShare` has already grown. Because `rewardDebt` is not set to `staked * accRewardPerShare` at entry, `harvest()` immediately pays out rewards that accrued before this user joined.

**Vulnerability type:** `incorrect_reward_accounting`

### 7. Reserve Updated Before Price Calculation (`incorrect_reserve_ordering`)

**Function to flag:** `buy()` in bonding-curve or AMM contracts.

**Pattern:** The incoming reserve amount is added to `reserveBalance` before computing the token output. The denominator in the price formula then uses the *new* (already-inflated) reserve, causing the contract to under-price the tokens and mint more than the bonding curve invariant allows.

**Vulnerability type:** `incorrect_reserve_ordering`

## False Positive Guard

Before reporting a finding, apply these filters to avoid noise:

- **Auction zero-start bids:** A minimum bid of 0 when `highestBid == 0` is standard auction initialization, not a vulnerability — the first bid simply sets the starting price.
- **Admin-controlled rate parameters used in refund/cancel calculations:** If the owner has explicit setter access and the rate change is transparent on-chain, a mismatch between subscription rate and refund rate is a design trade-off, not a exploitable bug unless there is a direct economic attack path with on-chain atomicity.
- **Governance snapshot at `block.number`:** Using the current block as a snapshot is a deliberate design choice in some governance systems (especially when the token already uses checkpointing via `getPastVotes`). Only flag when the snapshot is demonstrably not used during vote weight lookup.
- **Missing access control on grant creation when no existing-grant overwrite is possible:** If `createGrant` guards against duplicate grants (`require(grants[b].total == 0)`), the lack of an `onlyOwner` modifier may be intentional (community-driven vesting). Focus instead on whether the calculation inside `vested()` is correct.
