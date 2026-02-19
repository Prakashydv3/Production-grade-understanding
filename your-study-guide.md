# Your Study Guide: Blockchain Authority & Safety

**Purpose**: A personal reference to understand and review all blockchain concepts quickly.

---

## Quick Definitions

| Term | What It Is | Why It Matters |
|------|-----------|----------------|
| **Block** | A page in the ledger containing transactions | Blocks are chained; changing one breaks all after it |
| **Transaction** | A transfer of money/assets from one account to another | Must be authorized and must not double-spend |
| **Validator** | A computer that proposes and verifies blocks | Also called miner or proposer |
| **Finality** | The point where a block becomes permanent | Once finalized, it cannot be changed |
| **Nonce** | A sequence number on each account | Prevents the same transaction from executing twice |
| **State Root** | A cryptographic fingerprint of all accounts | If accounts are corrupted, state root changes |
| **Consensus** | Agreement among validators on which chain is correct | Requires 2/3+ validators to agree |
| **Fork** | A split in the blockchain (two competing chains) | Network eventually picks the heavier chain |
| **Authority** | The right to make a decision affecting the ledger | Distributed across validators, not centralized |
| **Gate** | A checkpoint that verifies input before allowing it | No transaction passes without going through gates |

---

## The Transaction Journey (5 Steps)

```
Step 1: Alice signs a transaction
        ↓ (Signature Gate)
Step 2: Transaction enters the mempool (waiting room)
        ↓ (Format Gate)
Step 3: Validator includes it in a block
        ↓ (Consensus Gate)
Step 4: 2/3 validators agree block is valid
        ↓ (Finality Gate)
Step 5: FINALIZED - Transaction is permanent
        ↓
Block added to chain, always, forever
```

---

## 7 Gates That Protect The System

### ✅ Gate 1: Signature Gate
**Checks**: Is the transaction really signed by the sender?
**Protects against**: Impersonation, unauthorized transfers
**How**: Cryptographic signature verification

### ✅ Gate 2: Nonce Gate
**Checks**: Is this the next transaction in sequence from this sender?
**Protects against**: Replay attacks (same transaction twice)
**How**: Transaction counter that increments after each use

### ✅ Gate 3: Fund Gate
**Checks**: Does the sender have enough money?
**Protects against**: Overdrafts, spending money you don't have
**How**: Check balance before deducting funds

### ✅ Gate 4: State Root Gate
**Checks**: After applying all transactions, do accounts match the promised state?
**Protects against**: Hidden state corruption
**How**: Cryptographic hash of all account balances

### ✅ Gate 5: Reward Gate
**Checks**: Did the validator create tokens only per protocol?
**Protects against**: Inflation, arbitrary money creation
**How**: Hardcoded reward schedule

### ✅ Gate 6: Finality Gate
**Checks**: Do 2/3 validators agree this block is final?
**Protects against**: Changing blocks after they're "done"
**How**: Consensus threshold check

### ✅ Gate 7: Chain Gate
**Checks**: Which chain is the real one if there are competing chains?
**Protects against**: One node taking over the network
**How**: Fork choice rule (follow the chain with most weight)

---

## 5 Authority Layers

### Layer 1: Validators (Consensus Authority)
**What they control**: Block creation and finalization
**How they're limited**: Requires 2/3 agreement
**Risk**: Validator collusion (mitigated by decentralization)

### Layer 2: Protocol (Supply Authority)
**What it controls**: How many tokens are created per block
**How it's limited**: Rules are hardcoded
**Risk**: Bug in reward logic (mitigated by code review)

### Layer 3: Cryptography (State Authority)
**What it controls**: Authorization (who can move money)
**How it's limited**: Only valid signatures pass
**Risk**: Private key compromise (mitigated by key security)

### Layer 4: Fork Choice Rule (Chain Authority)
**What it controls**: Which chain is canonical
**How it's limited**: Deterministic algorithm
**Risk**: Client malfunction (mitigated by testing)

### Layer 5: Network (Propagation Authority)
**What it controls**: Which messages are relayed
**How it's limited**: Only valid messages are forwarded
**Risk**: DoS attacks (mitigated by rate limiting)

---

## 5 Things That Must NEVER Happen

### 🚫 Never 1: Double-Spending
**What**: Alice sends same money to both Bob and Carol
**Why bad**: Money is created from nothing
**Prevention**: Nonce gate (sequence prevents reuse)

### 🚫 Never 2: Validator Privilege
**What**: Validator gives themselves extra coins
**Why bad**: Supply becomes uncontrolled
**Prevention**: Reward gate (hardcoded amount checked)

### 🚫 Never 3: Rewriting Finalized History
**What**: Network goes back and changes blocks
**Why bad**: History becomes unreliable
**Prevention**: Finality gate (old blocks cannot be changed)

### 🚫 Never 4: Silent State Corruption
**What**: Accounts are changed without anyone noticing
**Why bad**: Ledger becomes invalid
**Prevention**: State root gate (fingerprint changes if corrupted)

### 🚫 Never 5: Unauthorized Transactions
**What**: Alice's money moves without her permission
**Why bad**: Assets are stolen
**Prevention**: Signature gate (cryptographic authorization)

---

## Failure Scenarios & How They're Handled

| Failure | What Happens | What Breaks | What Stays Safe |
|---------|--------------|------------|-----------------|
| **One validator crashes** | 2 other validators continue | Block production slows 33% | Everything else (2/3 still agree) |
| **Network splits in half** | Group A (60%) & Group B (40%) both produce blocks | Group B's temporary blocks | Group A is canonical (more weight) |
| **Attacker proposes invalid block** | Other validators reject it | Attacker loses stake | Block never added to chain |
| **Replay attack attempted** | Same transaction sent twice | Would execute without nonce | Nonce prevents 2nd execution |
| **Data corruption in transit** | Block signature no longer matches | Block is malformed | Network rejects before relaying |

---

## Authority Creep: The Silent Risk

**What is it**: Someone gains control they weren't supposed to have.

**Why it's dangerous**:
- Silent (might not be noticed)
- Incremental (small changes add up)
- Irreversible (cannot be undone on live network)

### Example 1: Reward Mutation
- **Designed**: Protocol specifies 5 coins per block
- **Creep**: Code allows validator to change their reward
- **Result**: Validator gives themselves 1000 coins
- **Detection**: Check if reward schedule is hardcoded or mutable

### Example 2: Signature Bypass
- **Designed**: All transactions require signatures
- **Creep**: New code path: "Admin transactions skip signature"
- **Result**: Attacker sends unsigned transaction
- **Detection**: All code paths must verify signatures

### Example 3: Validator Selection Manipulation
- **Designed**: Slots assigned randomly
- **Creep**: Validator influences random seed
- **Result**: Validator predicts their own slots
- **Detection**: Check if seed is from finalized block (immutable)

---

## 10 Core Principles

### 1. Determinism is Sacred
Same input → Same output, always. No randomness in critical paths.

### 2. Authority is Distributed
No single person has absolute power. Consensus requires agreement.

### 3. No Silent Mutations
All changes are logged and verifiable. Nothing happens in secret.

### 4. Immutability of History
Finalized blocks are permanent. History cannot be rewritten.

### 5. Transparency Over Secrecy
All decisions can be audited. Admin backdoors are forbidden.

### 6. Gates Before State Changes
Every transaction passes through verification before changing accounts.

### 7. Nonces Prevent Replays
Same transaction cannot execute twice. Sequence numbers enforce order.

### 8. Supply is Algorithmic
Tokens are created per rules, not per whim. No discretionary minting.

### 9. Cryptography Proves Authorization
Private key is the only proof of ownership. No forgery is possible.

### 10. Consensus is the Source of Truth
More validators' agreement = more true. Decentralization prevents takeover.

---

## Your 8-Day Roadmap (What You're Doing)

```
PHASE 1 (Days 1-2): Structure Mapping ✅ DONE
  What: Where are blocks, state, supply stored?
  Files: node-lifecycle-map, state-ownership-map, deterministic-boundary-map

PHASE 2 (Days 2-3): Authority Isolation ✅ YOU ARE HERE
  What: Who decides what? Where could power expand?
  Files: authority-map, execution-gate-analysis, authority-creep-risk

PHASE 3 (Days 2-4): Failure Surface Modeling
  What: How can mainnet fail? What must never halt?
  Files: failure-surface-model, graceful-degradation-map, availability-rules

PHASE 4 (Days 4-5): Observability & Drift Detection
  What: What must be logged? How to detect corruption?
  Files: mainnet-observability, invariant-checklist, semantic-drift

PHASE 5 (Days 5-6): State Integrity & Immutability
  What: How is state protected? What guards supply?
  Files: mutation-risk-map, supply-integrity, immutability-report

PHASE 6 (Days 6-7): Replay Safety & Determinism
  What: How to guarantee replay consistency?
  Files: replay-safety-model, deterministic-reconstruction, nondeterminism-risks

PHASE 7 (Days 7-8): Mainnet Readiness Gate
  What: What is safe? What must never change?
  Files: readiness-declaration, red-line-declaration, continuity-bridge
```

---

## The Three Questions You Ask About Everything

When reading any code, ask yourself:

### Question 1: "Who decides this?"
**Why**: Identify who has authority and if it's appropriate

**Example**:
- "Who decides if a transaction is valid?" 
  → Signature gate + state machine
- "Who decides how many tokens to create?"
  → Protocol algorithm
- "Who decides which chain is real?"
  → Fork choice rule

### Question 2: "What could go wrong here?"
**Why**: Identify failure modes and risks

**Example**:
- "What if signature verification fails?"
  → Unsigned transaction would execute (bad)
- "What if nonce is not checked?"
  → Same transaction could execute twice (double-spend)
- "What if finality can be overridden?"
  → Blocks could be changed after being "done" (history rewrite)

### Question 3: "How is this protected?"
**Why**: Identify what prevents the wrong thing

**Example**:
- "How is double-spending prevented?"
  → Nonce counter prevents same nonce from using twice
- "How is unauthorized money transfer prevented?"
  → Signature gate prevents unsigned transactions
- "How is inflation prevented?"
  → Reward gate checks amount matches protocol

---

## One-Page Quick Reference

When you need to remember fast:

- **Gate 1-7**: Signature, Nonce, Fund, State Root, Reward, Finality, Chain ✅
- **Authority 5 layers**: Validators, Protocol, Crypto, Fork Choice, Network ✅
- **Never 5 things**: Double-spend, Validator privilege, Rewrite history, Silent corruption, Unauthorized tx ✅
- **Protect with**: Gates, Nonces, Signatures, Hardcoding, Consensus ✅
- **Detect with**: Logs, State roots, Signature verification, Finality checks ✅

---

## How to Use This File

1. **Before reading code**: Review the definitions and 7 gates
2. **While reading code**: Ask the 3 questions for each function
3. **After reading code**: Write down which authorities are involved
4. **When confused**: Check the failure scenarios table
5. **When risk-checking**: Review authority creep examples

---

## Your Success Criteria

By the end of Day 3, you should:

- [ ] Understand how blocks are made (Step 1-5 transaction journey)
- [ ] Know all 7 gates and what each protects
- [ ] Identify the 5 authority layers in any code
- [ ] Spot the 5 critical rules being enforced
- [ ] Detect potential authority creep
- [ ] Map failure scenarios and what mitigates them

If you can answer all 3 questions about any code system:
- "Who decides this?"
- "What could go wrong?"
- "How is it protected?"

**You are ready for Phase 3.**

---

## Remember This

You are not building a blockchain.
You are **reading** an existing blockchain.
You are not **innovating**.
You are **detecting** risks.

Every authority decision must be:
- ✅ Documented
- ✅ Bounded
- ✅ Verifiable
- ✅ Immutable

If it's not, it's a risk.

Find all the risks.

