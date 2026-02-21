# Nondeterminism Risk Register

## High-Risk Sources
- System time usage (CRITICAL)
- Random number generation (CRITICAL)
- Floating point arithmetic (CRITICAL)
- Map iteration order (HIGH)
- External API calls (CRITICAL)

## Medium-Risk Sources
- File system operations
- Network timing
- Concurrency
- Memory addresses
- Locale/timezone

## Low-Risk Sources
- Log timestamps (acceptable)
- Performance metrics (acceptable)
- Debug output (acceptable)

## Mitigation Strategies
- Pure functions for all consensus logic
- Deterministic testing
- Cross-node verification
- Code review checklist
- Static analysis

## Detection Methods
- State root comparison
- Parallel execution
- Cross-platform testing
- Historical replay
- Fuzzing

## Prevention Checklist
- No system time in consensus
- No random generation in consensus
- No floating point in consensus
- No unordered map iteration
- No external API calls
- All functions are pure