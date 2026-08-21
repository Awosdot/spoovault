# Release Conditions & Proof-of-Life Timers

Vaults can be configured to release documents automatically under specific circumstances, such as inactivity.

## Release Modes
1. **Anytime**: Instant access.
2. **LiveOnly**: Accessible only while the owner is actively verifying their presence.
3. **EmergencyOnly**: Accessible in emergency mode or if the owner is inactive.
4. **PostDeathOnly**: Unlocked strictly after the inactivity threshold is exceeded.

## Proof-of-Life Mechanism
- The vault creator records a "Proof of Life" on-chain (`proveLife` / `prove_life`).
- If no proof is registered for longer than the `inactivityPeriod` (e.g., 30 days), the vault enters a "post-death" status, allowing beneficiary requests.

## Block-Height Buffer
Because `block.timestamp` (EVM) and the ledger close time (Soroban) are ultimately reported by
the block producer, a "post-death" unlock that relies on the timestamp alone can be nudged
early by a miner/validator drifting the timestamp within the small window most chains
tolerate. To close that gap, post-death unlock requires a second, independent condition:
enough real blocks (`block.number`) or ledgers (`ledger().sequence()`) must also have elapsed
since the last proof of life.

- Each vault tracks `minBlockDelta`/`min_block_delta` (default: `DEFAULT_MIN_BLOCK_DELTA`
  blocks on EVM, `DEFAULT_MIN_LEDGER_DELTA` ledgers on Soroban) alongside the block/ledger
  number recorded at vault creation and at every `proveLife`/`prove_life` call.
- `configureBlockHeightBuffer` (EVM) / `configure_block_height_buffer` (Soroban) lets the
  vault creator adjust this buffer, bounded by `MAX_BLOCK_DELTA`/`MAX_LEDGER_DELTA`.
- Post-death unlock now requires **both** the timestamp threshold (`block.timestamp >=
  lastProofOfLife + inactivityPeriod`) **and** the block/ledger delta
  (`block.number >= lastProofOfLifeBlock + minBlockDelta`) to be satisfied — meeting only one
  of the two leaves the vault locked.