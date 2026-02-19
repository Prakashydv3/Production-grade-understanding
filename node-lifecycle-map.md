# Node Lifecycle Map

## Init

**Entry**: Node startup

**Actions**:
- Load genesis block
- Initialize storage backend
- Load cryptographic keys
- Parse configuration
- Establish network identity
- Initialize mempool
- Set up RPC endpoints

**State**: UNINITIALIZED → INITIALIZED

**Deterministic requirements**:
- Genesis hash must match network
- Configuration parsing must be reproducible
- Storage initialization must be idempotent

**Failure modes**:
- Missing genesis
- Corrupted storage
- Invalid keys
- Configuration error

## Sync

**Entry**: Node joins network

**Actions**:
- Discover peers
- Request block headers
- Download blocks
- Verify signatures
- Validate transactions
- Rebuild state from genesis
- Catch up to chain tip

**State**: INITIALIZED → SYNCING → SYNCED

**Deterministic requirements**:
- Block validation must be pure
- State reconstruction must be identical across nodes
- Transaction ordering must be preserved
- Hash verification must be exact

**Failure modes**:
- Network partition
- Peer unavailability
- Invalid block received
- State reconstruction failure
- Disk exhaustion

## Validate

**Entry**: New block or transaction received

**Actions**:
- Verify block structure
- Verify signatures
- Check transaction nonces
- Verify state transitions
- Check consensus rules
- Validate rewards
- Verify stake requirements

**State**: SYNCED → VALIDATING → VALID/INVALID

**Deterministic requirements**:
- Same input → same output
- No external state dependencies
- No timestamp dependencies (except explicit rules)
- No randomness

**Failure modes**:
- Invalid signature
- Double spend
- Insufficient stake
- Invalid state transition
- Consensus violation

## Mine

**Entry**: Node selected as validator

**Actions**:
- Select transactions from mempool
- Order transactions deterministically
- Calculate state transitions
- Compute block hash
- Sign block
- Calculate rewards
- Update stake ledger
- Prepare broadcast

**State**: VALIDATOR_SELECTED → MINING → BLOCK_CREATED

**Deterministic requirements**:
- Transaction ordering must be deterministic
- State transition must be pure
- Reward calculation must be deterministic
- Block hash must be reproducible
- No external randomness

**Failure modes**:
- Transaction selection error
- State transition failure
- Signature failure
- Reward calculation error

## Broadcast

**Entry**: Block or transaction ready

**Actions**:
- Propagate to peers
- Handle gossip protocol
- Manage broadcast queue
- Track propagation
- Handle rebroadcast

**State**: BLOCK_CREATED → BROADCASTING → BROADCAST_COMPLETE

**Deterministic requirements**:
- Message serialization must be deterministic
- Duplicate detection must be deterministic
- Broadcast order does not affect consensus

**Failure modes**:
- Network partition
- Peer disconnection
- Message loss
- Propagation delay

## Shutdown

**Entry**: Termination signal

**Actions**:
- Flush pending transactions
- Close network connections
- Persist state to disk
- Close database
- Release resources
- Log final state

**State**: ANY → SHUTTING_DOWN → SHUTDOWN

**Deterministic requirements**:
- State persistence must be atomic
- Shutdown must be idempotent
- Restart must recover consistent state

**Failure modes**:
- Incomplete flush
- Corrupted database
- Resource leak
- Unclean shutdown

## Cross-Phase Invariants

- State transitions are append-only
- Validation is deterministic
- Authority is explicit
- Network variance does not affect consensus
