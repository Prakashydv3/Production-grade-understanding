# Deterministic Boundary Map

## Deterministic Zone (Must Be Pure)

### Block Validation

**Function**: ValidateBlock(block) → valid/invalid

**Determinism**: ABSOLUTE

**Inputs**: Block header, transactions, previous state root

**Output**: Boolean

**Environment isolation**:
- No system time reads
- No network state reads
- No file system reads
- No random sources

**Failure if non-deterministic**: Network fork, consensus failure

### Transaction Validation

**Function**: ValidateTransaction(tx, state) → valid/invalid

**Determinism**: ABSOLUTE

**Inputs**: Transaction data, current state

**Output**: Boolean + error

**Environment isolation**:
- No timestamp dependencies
- No external data sources
- Pure cryptographic operations

**Failure if non-deterministic**: Double spend, state divergence

### State Transition

**Function**: ApplyTransaction(tx, state) → newState

**Determinism**: ABSOLUTE

**Inputs**: Transaction, current state

**Output**: New state

**Environment isolation**:
- No network access
- No file system access
- No randomness
- No system time

**Failure if non-deterministic**: State divergence, consensus failure

### Reward Calculation

**Function**: CalculateReward(blockHeight, transactions) → reward

**Determinism**: ABSOLUTE

**Inputs**: Block height, transaction fees, consensus parameters

**Output**: Reward amount

**Environment isolation**:
- No market data
- No external price feeds
- No randomness

**Failure if non-deterministic**: Supply divergence, validator disputes

### Block Hash Calculation

**Function**: CalculateBlockHash(block) → hash

**Determinism**: ABSOLUTE

**Inputs**: Block header, transactions, merkle root

**Output**: Block hash

**Environment isolation**: Pure hash function

**Failure if non-deterministic**: Chain verification failure, network split

### Transaction Ordering

**Function**: OrderTransactions(txSet) → orderedTxList

**Determinism**: ABSOLUTE

**Inputs**: Transaction set

**Output**: Ordered list

**Environment isolation**:
- No timestamp-based ordering (unless in transaction)
- No arrival-time ordering
- No priority randomization

**Failure if non-deterministic**: Different nodes produce different blocks

### Stake Selection

**Function**: SelectValidator(blockHeight, stakeSet) → validator

**Determinism**: ABSOLUTE

**Inputs**: Block height, stake distribution, deterministic seed

**Output**: Selected validator

**Environment isolation**:
- Seed from blockchain state only
- No external random sources
- No time-based selection

**Failure if non-deterministic**: Multiple validators claim same block

## Non-Deterministic Zone (Environment-Sensitive)

### Peer Discovery

**Function**: DiscoverPeers() → peerList

**Determinism**: NONE

**Why acceptable**: Does not affect consensus or state

### Block Propagation

**Function**: BroadcastBlock(block)

**Determinism**: NONE

**Why acceptable**: Does not affect block validity or state

### Mempool Management

**Function**: AddToMempool(tx)

**Determinism**: NONE

**Why acceptable**: Mempool is not consensus-critical

### Logging

**Function**: Log(message)

**Determinism**: NONE

**Why acceptable**: Logs are observability only

### Performance Metrics

**Function**: RecordMetric(name, value)

**Determinism**: NONE

**Why acceptable**: Metrics are observability only

### Sync Strategy

**Function**: SelectSyncPeers() → peers

**Determinism**: NONE

**Why acceptable**: Does not affect final state

## Boundary Violations (Critical Risks)

### Risk 1: Time-Based Validation

**Violation**: Using system time in validation

**Example**: `if (currentTime() > deadline) reject()`

**Why dangerous**: Nodes have clock skew, different results

**Mitigation**: Use block timestamp only

### Risk 2: External Data in State Transition

**Violation**: Reading external data during state transition

**Example**: `price = fetchFromOracle()`

**Why dangerous**: External data varies by node

**Mitigation**: External data must be in transactions

### Risk 3: Random Number Generation

**Violation**: Non-deterministic randomness

**Example**: `validator = selectRandom(validators)`

**Why dangerous**: Each node gets different result

**Mitigation**: Use deterministic seed from blockchain

### Risk 4: Floating Point Arithmetic

**Violation**: Floating point in consensus calculations

**Example**: `reward = blockReward * 0.1`

**Why dangerous**: Not deterministic across platforms

**Mitigation**: Use integer arithmetic only

### Risk 5: Hash Map Iteration

**Violation**: Iterating over unordered map

**Example**: `for (tx in transactionMap) process(tx)`

**Why dangerous**: Iteration order not guaranteed

**Mitigation**: Sort keys before iteration

## Enforcement

**Code review checklist**:
- [ ] No system time in deterministic zone
- [ ] No external data in deterministic zone
- [ ] No random generation in deterministic zone
- [ ] No floating point in consensus
- [ ] No unordered iteration in deterministic zone
- [ ] All state transitions are pure
- [ ] All validation logic is pure

**Testing requirements**:
- Same inputs must produce same outputs
- Tests must run on different platforms
- Verify byte-level equality
