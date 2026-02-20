# State Mutation Risk Map

## Block Addition Path

### Entry Point
Function: AddBlock(block)

### Mutation Sequence
1. Validate block structure
2. Verify block signature
3. Process transactions
4. Update account balances
5. Calculate new state root
6. Append block to chain
7. Update chain tip pointer

### Protecting Invariants

**Invariant B1**: Blocks are append-only
- Protection: No delete or modify operations
- Verification: Block hash chain integrity
- Bypass consequence: History rewrite, consensus failure

**Invariant B2**: Block height is sequential
- Protection: Height = previous height + 1
- Verification: Check height continuity
- Bypass consequence: Chain discontinuity, fork

**Invariant B3**: State root matches calculated state
- Protection: Recompute and compare
- Verification: Hash comparison
- Bypass consequence: State divergence across nodes

### External Input Influence

**Block data**: Provided by validator
- Risk: Malicious validator provides invalid block
- Mitigation: Multi-validator verification

**Network timing**: Block arrival time
- Risk: Timing affects processing order
- Mitigation: Block timestamp in consensus data

**Disk state**: Previous blocks and state
- Risk: Corrupted storage
- Mitigation: Hash verification on read

### Bypass Scenarios

**Scenario 1**: Direct database write
- Attacker bypasses AddBlock function
- Writes block directly to storage
- State root not updated
- Detection: State root mismatch

**Scenario 2**: Validation skip
- Code path skips validation
- Invalid block accepted
- Consensus breaks
- Detection: Other nodes reject block

**Scenario 3**: State calculation error
- Bug in state transition logic
- Wrong balances calculated
- State root mismatch
- Detection: Cross-node comparison

## Reward Minting Path

### Entry Point
Function: MintReward(blockHeight, validator)

### Mutation Sequence
1. Calculate reward amount
2. Verify validator eligibility
3. Create reward transaction
4. Update validator balance
5. Update total supply
6. Record in block

### Protecting Invariants

**Invariant R1**: Reward follows schedule
- Protection: Hardcoded reward formula
- Verification: Compare to expected amount
- Bypass consequence: Supply inflation/deflation

**Invariant R2**: One reward per block
- Protection: Reward minted only in AddBlock
- Verification: Count rewards per block
- Bypass consequence: Supply inflation

**Invariant R3**: Only validator receives reward
- Protection: Validator address verification
- Verification: Check recipient is block proposer
- Bypass consequence: Unauthorized minting

### External Input Influence

**Block height**: Determines reward amount
- Risk: Height manipulation
- Mitigation: Height verified by consensus

**Validator address**: Reward recipient
- Risk: Address spoofing
- Mitigation: Cryptographic verification

**Transaction fees**: Added to base reward
- Risk: Fee calculation error
- Mitigation: Deterministic fee calculation

### Bypass Scenarios

**Scenario 1**: Direct balance increase
- Attacker bypasses MintReward
- Increases balance directly
- Supply total not updated
- Detection: Supply audit mismatch

**Scenario 2**: Reward formula modification
- Code change increases reward
- Supply inflates faster
- Detection: Supply growth rate analysis

**Scenario 3**: Multiple reward claims
- Validator claims reward twice
- Supply inflates
- Detection: Reward count per block

## Stake Ledger Mutation Path

### Entry Point
Function: UpdateStake(validator, amount, operation)

### Mutation Sequence
1. Verify stake transaction signature
2. Check account balance
3. Lock/unlock tokens
4. Update stake ledger
5. Update validator set if needed
6. Record state change

### Protecting Invariants

**Invariant K1**: Stake ≤ account balance
- Protection: Balance check before staking
- Verification: Compare stake to balance
- Bypass consequence: Negative balance

**Invariant K2**: Unstaking follows time-lock
- Protection: Timestamp verification
- Verification: Check unlock time
- Bypass consequence: Instant unstaking, security risk

**Invariant K3**: Validator set updates are authorized
- Protection: Consensus-based updates
- Verification: Multi-validator agreement
- Bypass consequence: Unauthorized validator addition

### External Input Influence

**Stake transaction**: User-provided
- Risk: Invalid stake amount
- Mitigation: Balance verification

**Time-lock period**: Protocol parameter
- Risk: Time manipulation
- Mitigation: Block timestamp used

**Validator eligibility**: Minimum stake requirement
- Risk: Requirement bypass
- Mitigation: Threshold check enforced

### Bypass Scenarios

**Scenario 1**: Stake without balance
- Attacker stakes more than they have
- Negative balance created
- Detection: Balance verification

**Scenario 2**: Instant unstaking
- Attacker bypasses time-lock
- Withdraws immediately
- Security compromised
- Detection: Time-lock verification

**Scenario 3**: Validator set manipulation
- Attacker adds unauthorized validator
- Consensus compromised
- Detection: Validator set audit

## Token Registry Mutation Path

### Entry Point
Function: UpdateTokenRegistry(tokenId, metadata)

### Mutation Sequence
1. Verify update authorization
2. Validate token metadata
3. Update registry entry
4. Emit registry change event
5. Update state root

### Protecting Invariants

**Invariant T1**: Token creation is authorized
- Protection: Creator signature verification
- Verification: Check authorization
- Bypass consequence: Unauthorized tokens

**Invariant T2**: Token metadata is immutable
- Protection: No update after creation
- Verification: Check modification attempts
- Bypass consequence: Token properties change

**Invariant T3**: Token supply is tracked
- Protection: Supply counter per token
- Verification: Sum of balances = supply
- Bypass consequence: Supply mismatch

### External Input Influence

**Token metadata**: Creator-provided
- Risk: Invalid or malicious metadata
- Mitigation: Metadata validation

**Authorization proof**: Signature
- Risk: Signature forgery
- Mitigation: Cryptographic verification

### Bypass Scenarios

**Scenario 1**: Unauthorized token creation
- Attacker creates token without authorization
- Registry polluted
- Detection: Authorization verification

**Scenario 2**: Metadata modification
- Attacker changes immutable metadata
- Token properties altered
- Detection: Immutability check

**Scenario 3**: Supply manipulation
- Attacker mints tokens without authorization
- Supply inflates
- Detection: Supply audit

## Cross-Path Risks

### Risk 1: Race Conditions
- Multiple paths modify same state
- Concurrent updates conflict
- Mitigation: Atomic operations, locks

### Risk 2: Ordering Dependencies
- Path B depends on Path A completion
- Out-of-order execution breaks invariants
- Mitigation: Sequential processing

### Risk 3: Partial Failures
- Path completes partially
- State left inconsistent
- Mitigation: Atomic transactions, rollback

## Verification Procedures

### Continuous Verification
- State root after each block
- Supply total after each block
- Stake ledger after each epoch
- Token registry after each update

### Periodic Audits
- Full state reconstruction from genesis
- Supply calculation verification
- Stake ledger integrity check
- Token registry consistency check

### Alert Triggers
- State root mismatch
- Supply calculation error
- Negative balance detected
- Unauthorized state change