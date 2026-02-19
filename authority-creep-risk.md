# Authority Creep Risk Register

## Mission
Identify all points where authority could silently expand beyond its intended scope. Map governance risks. Prevent scope takeover. Production-grade risk analysis.

---

## What is Authority Creep?

**Authority creep** is when a decision-point gains power beyond its designed scope through:
1. Code path not in specification
2. Gate bypass (intentional or accidental)
3. Scope expansion in related function
4. Silent assumption change

**Risk level**: Critical
**Detection**: Diff review, behavior monitoring
**Prevention**: Specification enforcement

---

## Risk Domain 1: Consensus Authority Expansion

### Risk 1.1: Validator Set Modification Outside Protocol

**Creep Pattern**: Validator set changes are decided in consensus layer, but...

**Hidden authority vector**:
- Validator set stored in state
- State is mutable via transactions
- What if transaction can modify validator set?

**Scope expansion scenario**:
1. Designed authority: Only consensus can change validators
2. Hidden path: Any transaction that writes to validator set storage
3. Result: Authority expanded from validators to transaction signers

**Red flag**:
```
if (tx.recipient == VALIDATOR_SET_ADDRESS):
    apply_state_change(tx)  // Anyone can write
```

**Detection**:
- Search: "validator", "set", "modify", "add", "remove"
- Find: Storage address for validator set
- Check: Who can write to it?
- Expected: Only consensus logic
- Risk: Transaction logic can write to it

**Prevention**:
- Validator set must be stored in privileged location
- Only consensus code can modify it
- Transactions cannot write to it

**Audit checklist**:
- [ ] Validator set address is hardcoded
- [ ] Transactions cannot address validator set
- [ ] Only consensus functions modify validator set
- [ ] No transaction can become a validator

---

### Risk 1.2: Slot Assignment Manipulation

**Creep Pattern**: Block proposer slot is calculated, but...

**Hidden authority vector**:
- Slot is deterministic given (height, round, seed)
- Seed is derived from previous block
- What if previous block proposer can control seed?

**Scope expansion scenario**:
1. Designed authority: Protocol assigns slots
2. Hidden path: Validator can influence seed
3. Result: Validator can predict/influence own future slots

**Red flag**:
```
seed = hash(previous_block.data)
// If validator controls previous_block content, they influence seed
next_proposer = validators[seed % validator_count]
```

**Detection**:
- Find: Seed derivation logic
- Check: What influences the seed?
- Expected: Only previous block hash (immutable)
- Risk: Validator controls previous block content

**Prevention**:
- Seed must be derived only from committed data
- Committed data must be immutable
- No validator can influence committed seed

**Audit checklist**:
- [ ] Seed is derived from finalized block
- [ ] Validator cannot modify finalized blocks
- [ ] Slot assignment is deterministic
- [ ] No validator can engineer their own slot

---

### Risk 1.3: Fork Choice Rule Modification

**Creep Pattern**: Fork choice is rule-based, but...

**Hidden authority vector**:
- Fork choice code is client code
- Client code can be updated
- What if network fragment updates differently?

**Scope expansion scenario**:
1. Designed authority: Fork choice rule is immutable protocol
2. Hidden path: Different clients run different versions
3. Result: Authority expanded to client implementers

**Red flag**:
```
// Old fork choice
if (block_a_weight > block_b_weight):
    return block_a

// New fork choice (after update)
if (client_id == "special"):
    return block_a_always
```

**Detection**:
- Find: Fork choice algorithm
- Check: Can it be modified without consensus?
- Expected: Requires protocol upgrade
- Risk: Client can unilaterally change it

**Prevention**:
- Fork choice must be deterministic
- Changes must be network-wide consensus
- Version coordination protocol required

**Audit checklist**:
- [ ] Fork choice algorithm is documented
- [ ] Client implements exact specification
- [ ] No client preference logic
- [ ] Version coordination enforced

---

## Risk Domain 2: Supply Authority Expansion

### Risk 2.1: Reward Schedule Mutation

**Creep Pattern**: Rewards are on schedule, but...

**Hidden authority vector**:
- Reward schedule is stored in state
- State is updated via transactions
- What if transaction can changeRewardSchedule?

**Scope expansion scenario**:
1. Designed authority: Reward schedule is immutable protocol
2. Hidden path: Reward schedule is stored in state
3. Result: Authority expanded to state mutators

**Red flag**:
```
REWARD_SCHEDULE = stored_in_state  // Mutable
validator_reward = REWARD_SCHEDULE[block_height]
```

**Detection**:
- Find: Where is reward schedule stored?
- Expected: Hardcoded in protocol
- Risk: Stored in state (mutable via transaction)

**Prevention**:
- Reward schedule must be hardcoded
- No transaction can modify it
- Changes require protocol upgrade

**Audit checklist**:
- [ ] Reward schedule is constants in code
- [ ] Not stored in mutable state
- [ ] Immutable across runtime
- [ ] Cannot be overridden by transactions

---

### Risk 2.2: Minting Authority Scope

**Creep Pattern**: Only validators can trigger minting, but...

**Hidden authority vector**:
- Minting happens after block finalization
- What if non-validator can finalize blocks?
- Then non-validator can trigger minting

**Scope expansion scenario**:
1. Designed authority: Validators mint rewards
2. Hidden path: Finalization is used to trigger minting
3. Result: Authority expanded to anyone who can trigger finalization

**Red flag**:
```
if (block_finalized):
    mint_reward(validator, REWARD_AMOUNT)
    // If non-validator can create finalized blocks, they mint
```

**Detection**:
- Find: Minting logic
- Check: What condition triggers it?
- Expected: Only block_finalized from validators
- Risk: Finalization condition can be triggered by non-validators

**Prevention**:
- Block finalization only from validators
- Minting only after consensus finality gate
- No alternative path to finalization

**Audit checklist**:
- [ ] Only validators can propose blocks
- [ ] Only consensus can finalize
- [ ] Only finalized blocks trigger minting
- [ ] No bypass to finalization

---

### Risk 2.3: Supply Total Overflow

**Creep Pattern**: Supply increases are controlled, but...

**Hidden authority vector**:
- Supply total is an integer variable
- What if reward is malformed and causes overflow?
- Overflow wraps supply to 0 or small number

**Scope expansion scenario**:
1. Designed authority: Protocol defines correct supply
2. Hidden path: Integer overflow in supply addition
3. Result: Supply can be secretly reset

**Red flag**:
```
supply_total += reward_amount
// If supply_total is u64 and nears max value, wraps to 0
```

**Detection**:
- Find: Supply minting code
- Check: Integer type used
- Expected: Safe arithmetic (BigInt or overflow check)
- Risk: Native integer with silent overflow

**Prevention**:
- Use safe integer arithmetic
- Check for overflow before minting
- Reject block if overflow would occur

**Audit checklist**:
- [ ] Safe integer type used for supply
- [ ] Overflow checking enabled
- [ ] Block rejected if supply would overflow
- [ ] No silent supply loss

---

## Risk Domain 3: State Mutation Authority Expansion

### Risk 3.1: Signature Verification Bypass

**Creep Pattern**: Transactions require signatures, but...

**Hidden authority vector**:
- Signature verification is a function
- What if alternative code path skips it?
- Non-signed transactions execute

**Scope expansion scenario**:
1. Designed authority: Only signed transactions execute
2. Hidden path: Admin transaction bypass signature check
3. Result: Authority expanded to anyone submitting admin transactions

**Red flag**:
```
if (tx.admin_type):
    apply_state_change(tx)  // No signature check!
else:
    if verify_signature(tx):
        apply_state_change(tx)
```

**Detection**:
- Find: Transaction validation code
- Check: All paths require signatures?
- Expected: 100% of paths verify signature
- Risk: Any code path that skips signature

**Prevention**:
- Signature verification is mandatory
- No exception cases
- All transactions require authorization

**Audit checklist**:
- [ ] No code path skips signature
- [ ] No transaction type bypasses auth
- [ ] Signature is first check always
- [ ] Admin operations still require signature

---

### Risk 3.2: Nonce Bypass

**Creep Pattern**: Nonces prevent replay, but...

**Hidden authority vector**:
- Nonce check is in transaction validation
- What if state transition skips nonce check?
- Same transaction applies twice

**Scope expansion scenario**:
1. Designed authority: Nonce prevents double execution
2. Hidden path: State machine doesn't check nonce
3. Result: Authority expanded to replay attackers

**Red flag**:
```
// In transaction validation
check_nonce(tx)

// In state machine (called later)
apply_transfer(tx.sender, tx.recipient, tx.amount)
// No nonce increment! Can replay
```

**Detection**:
- Find: State transition code
- Check: Where is nonce incremented?
- Expected: Must increment after each transaction
- Risk: Nonce not incremented in state code

**Prevention**:
- Nonce increment is mandatory in state transition
- Nonce is checked before state mutation
- Both validation and execution maintain nonce

**Audit checklist**:
- [ ] Nonce validated in transaction layer
- [ ] Nonce incremented in state layer
- [ ] Nonce cannot be incremented twice
- [ ] Nonce cannot be skipped

---

### Risk 3.3: Authorization Check Relocation

**Creep Pattern**: Authorization is verified first, but...

**Hidden authority vector**:
- Authorization logic is moved to different function
- New function is not always called
- Old authorization path still exists

**Scope expansion scenario**:
1. Designed authority: Authorization is checked in ValidateTransaction()
2. Hidden path: Someone refactors check to new function
3. Result: Old ValidateTransaction() has no check; can be bypassed

**Red flag**:
```
// Old code
function ValidateTransaction(tx):
    verify_signature(tx)
    apply_transaction(tx)

// Refactored code
function ValidateTransaction(tx):
    apply_transaction(tx)  // Check moved to new function

function verify_authorization(tx):
    verify_signature(tx)  // But not always called
```

**Detection**:
- Find: Where authorization is checked
- Check: All transaction paths through that check?
- Expected: Single entry point for all transactions
- Risk: Authorization logic is decentralized

**Prevention**:
- Single transaction validation gateway
- All transactions pass through one function
- Authorization is first operation

**Audit checklist**:
- [ ] Single entry point for transaction validation
- [ ] Authorization is checked immediately
- [ ] No transaction can bypass entry point
- [ ] All code paths converge at validation

---

## Risk Domain 4: Finality Authority Expansion

### Risk 4.1: Finality Threshold Lowering

**Creep Pattern**: Finality requires 2/3 signatures, but...

**Hidden authority vector**:
- Finality threshold is a constant
- What if constant is changed to 1/3?
- Finality achieved with fewer validators

**Scope expansion scenario**:
1. Designed authority: 2/3 validators must agree
2. Hidden path: Code changes threshold to 1/3
3. Result: Authority expanded to smaller validator subset

**Red flag**:
```
const FINALITY_THRESHOLD = 67  // 2/3

// Updated in code
const FINALITY_THRESHOLD = 34  // 1/3 (bug or intentional?)
```

**Detection**:
- Find: Finality threshold constant
- Check: Is it immutable after network launch?
- Expected: Never changes
- Risk: Changed in deploy

**Prevention**:
- Finality threshold is network parameter
- Immutable for network lifetime
- Changes require hard fork and consensus

**Audit checklist**:
- [ ] Finality threshold hardcoded
- [ ] Cannot be modified in code update
- [ ] Network genesis locks threshold
- [ ] No changing threshold mid-network

---

### Risk 4.2: Early Finality Claims

**Creep Pattern**: Blocks are finalized when threshold reached, but...

**Hidden authority vector**:
- Finality detection runs in each client
- What if client finalizes before consensus is confirmed?
- Client assumes finality; reality is tentative

**Scope expansion scenario**:
1. Designed authority: Network consensus finalizes blocks
2. Hidden path: Client finalizes based on local observation
3. Result: Authority expanded to client's optimism bias

**Red flag**:
```
if (self.signatures_received >= threshold):
    self.finalize(block)  // Local decision, not network consensus
```

**Detection**:
- Find: Finality check code
- Check: Does it wait for network confirmation?
- Expected: Confirmed from multiple sources
- Risk: Finalized on local observation only

**Prevention**:
- Finality only after network finalization confirmed
- Cannot finalize unilaterally
- Must receive confirmation from network

**Audit checklist**:
- [ ] Finality is network event, not local observation
- [ ] Client waits for finality confirmations
- [ ] Does not assume finality prematurely
- [ ] Can detect and recover from finality failure

---

## Risk Domain 5: Network Authority Expansion

### Risk 5.1: Message Relay Criteria Expansion

**Creep Pattern**: Only valid messages are relayed, but...

**Hidden authority vector**:
- Relay logic is in node code
- What if relay decision can be influenced by client?
- Client can selectively relay

**Scope expansion scenario**:
1. Designed authority: Network relays all valid messages
2. Hidden path: Client adds filter to relay logic
3. Result: Authority expanded to node operator

**Red flag**:
```
if message.valid and message.from_trusted_peer:
    relay(message)  // Can suppress from non-trusted peers
```

**Detection**:
- Find: Message relay decision logic
- Check: Based only on message validity?
- Expected: Yes, valid messages are always relayed
- Risk: Relay depends on sender or other factors

**Prevention**:
- Message relay based solely on message validity
- No sender-based filtering
- All valid messages propagate

**Audit checklist**:
- [ ] Relay is based on message validity only
- [ ] No sender-based filtering
- [ ] No operator preference logic
- [ ] DoS protection is separate from relay logic

---

### Risk 5.2: Message Rate Limiting Abuse

**Creep Pattern**: Rate limiting prevents DoS, but...

**Hidden authority vector**:
- Rate limit is per-sender
- What if node operator sets limit very low?
- Only connected peers can communicate

**Scope expansion scenario**:
1. Designed authority: Rate limiting prevents DoS
2. Hidden path: Operator sets limit to 1 msg/hour
3. Result: Authority expanded to operator's network control

**Red flag**:
```
rate_limit = 1_message_per_second
// Changed to
rate_limit = 1_message_per_hour  // Blockade of competing peer
```

**Detection**:
- Find: Rate limit configuration
- Check: Can operator change it?
- Expected: Default is reasonable
- Risk: Operator can set arbitrarily low

**Prevention**:
- Rate limit has reasonable defaults
- Rate limit is network parameter
- Cannot be set below safety threshold

**Audit checklist**:
- [ ] Rate limit defaults are reasonable
- [ ] Cannot be lowered below safety threshold
- [ ] Network consensus can adjust if needed
- [ ] Operator cannot use limits for censorship

---

## Authority Creep Detection Checklist

### For Each Function:

- [ ] **Specification**: What is intended authority scope?
- [ ] **Code**: What is actual authority scope?
- [ ] **Match**: Do specification and code match exactly?
- [ ] **Bypass**: What code paths could bypass the gate?
- [ ] **Expansion**: What functions could expand this authority?
- [ ] **Mutation**: What could change this authority at runtime?

### For Each Data Structure:

- [ ] **Storage**: Where is it stored?
- [ ] **Mutation**: Who can modify it?
- [ ] **Expected**: Who should be able to modify it?
- [ ] **Mismatch**: Where does expected !== actual?
- [ ] **Dependency**: What depends on this data?
- [ ] **Risk**: If data is corrupted, what fails?

### For Each Authority Layer:

- [ ] **Validators**: Can non-validators become validators?
- [ ] **Supply**: Can supply be minted outside protocol?
- [ ] **State**: Can state be mutated without signature?
- [ ] **Finality**: Can finality be overridden?
- [ ] **Network**: Can network be partitioned by one node?

---

## Authority Creep Mitigation Strategy

### Layer 1: Code Review
- Code diff review against specification
- Every change checked for scope expansion
- Authority boundaries explicitly marked in code

### Layer 2: Testing
- Test suite includes authority bypass attempts
- Fuzz testing on gate functions
- Adversarial testing on boundary conditions

### Layer 3: Runtime Monitoring
- Log all authority decisions
- Flag unusual authority patterns
- Produce alert on scope expansion

### Layer 4: Formal Verification
- Specify authority mathematically
- Prove code matches specification
- Detect scope expansion formally

### Layer 5: Governance
- Authority changes require multiple signers
- Network upgrade required for scope changes
- Long-term commitment to authority model

---

## Summary: Authority Creep is the #1 Risk

- It is silent (might not be noticed)
- It is incremental (small changes accumulate)
- It is systemic (affects whole ledger)
- It is unrecoverable (cannot be undone on live network)

**Every authority decision must be guarded against creep.**

