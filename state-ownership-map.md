# State Ownership Map

## Block Storage

**Location**: Persistent disk

**Contents**:
- Block headers
- Block bodies
- Transactions
- Signatures
- Timestamps

**Ownership**: Immutable once written

**Access**: Write-once, read-many

**Persistence**: Must survive restart

**Integrity**: Hash chain

**Mutation rules**:
- Blocks cannot be modified
- Blocks cannot be deleted
- Only append allowed

## State Database

**Location**: Key-value store

**Contents**:
- Account balances
- Contract state
- Stake allocations
- Validator set
- Nonce counters

**Ownership**: Derived from blocks

**Access**: Read-write with versioning

**Persistence**: Must be reconstructible

**Integrity**: State root hash

**Mutation rules**:
- Changes only via valid transactions
- Must be deterministically derivable
- State root must match block header

## Mempool

**Location**: In-memory

**Contents**:
- Pending transactions
- Transaction metadata
- Priority queue

**Ownership**: Ephemeral, node-local

**Access**: Read-write, volatile

**Persistence**: None

**Integrity**: Signature verification

**Mutation rules**:
- Transactions can be added
- Transactions can be removed
- No ordering guarantees across nodes

## Chain Metadata

**Location**: Persistent storage

**Contents**:
- Chain tip pointer
- Fork choice data
- Peer information
- Sync status

**Ownership**: Node-local, mutable

**Access**: Read-write

**Persistence**: Should survive restart

**Integrity**: Consistency checks

**Mutation rules**:
- Updated during operation
- Must remain consistent with blocks
- Can be reconstructed if corrupted

## Supply Tracking

**Location**: Derived state

**Calculation**: Genesis Supply + Sum(Block Rewards) - Sum(Burns)

**Integrity invariants**:
- Must be calculable from block history
- Cannot decrease (unless explicit burns)
- Reward minting follows deterministic rules
- No supply without corresponding block

**Verification**: Recompute from genesis, must match exactly

## Stake Tracking

**Location**: Derived state

**Structure**:
```
validator_address: {amount, timestamp, status}
```

**Integrity invariants**:
- Changes only via explicit transactions
- Staked amount cannot exceed balance
- Unstaking follows time-lock rules
- Slashing recorded immutably

**Verification**: Recompute from stake transactions

## State Reconstruction

**Process**:
1. Start from genesis
2. Apply block transactions
3. Compute state root
4. Verify against block header
5. Repeat for all blocks

**Determinism**: Same blocks → same state

**Failure detection**:
- State root mismatch
- Supply calculation error
- Negative balances

## Persistence Boundaries

**Must persist**:
- All blocks
- All transactions
- Current state root
- Supply total
- Stake ledger

**May persist**:
- Mempool contents
- Peer connections
- Metrics
- Logs

**Must not persist**:
- Temporary computation state
- Network buffers
- Unvalidated data
