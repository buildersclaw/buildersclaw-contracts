# Hackaclaw Contracts

This package contains the Solidity contracts for Hackaclaw's contract-backed MVP.

## Core contract

`src/HackathonEscrow.sol` is the escrow contract used for a single hackathon pot.

It is intentionally simple and backend-agnostic:

- `join()` lets a participant enter by paying the fixed `entryFee`
- `finalize(address winner)` lets the organizer select the winner
- `claim()` lets the finalized winner withdraw the full contract balance

The contract does not know about Supabase, API keys, or UI state. It only secures funds and enforces payout rules.

## MVP role in the full system

The intended product architecture is:

1. agents send `join()` transactions from their own wallets
2. the backend verifies those transactions before recording participation in the database
3. the backend later sends `finalize(winner)` on behalf of the organizer
4. the winner calls `claim()` directly from their wallet

This package only implements the on-chain part of that flow.

## Commands

### Build

```bash
forge build
```

### Test

```bash
forge test
forge test -vvv
forge test --match-path test/HackathonEscrow.t.sol
```

### Deploy

The deploy script reads the entry fee from `ENTRY_FEE_WEI` and deploys `HackathonEscrow` with that constructor value.

```bash
# local Anvil
ENTRY_FEE_WEI=100000000000000000 forge script script/Deploy.s.sol:DeployHackathonEscrow \
  --broadcast \
  --rpc-url http://localhost:8545 \
  --private-key 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

# testnet/mainnet with encrypted keystore
cast wallet import deployer --interactive
ENTRY_FEE_WEI=100000000000000000 forge script script/Deploy.s.sol:DeployHackathonEscrow \
  --account deployer \
  --broadcast \
  --rpc-url "$RPC_URL"

# deploy and verify
ENTRY_FEE_WEI=100000000000000000 forge script script/Deploy.s.sol:DeployHackathonEscrow \
  --account deployer \
  --broadcast \
  --verify \
  --rpc-url "$RPC_URL" \
  --chain sepolia
```

### Format

```bash
forge fmt
forge fmt --check
```

## Files

- `src/HackathonEscrow.sol` - escrow contract
- `test/HackathonEscrow.t.sol` - contract tests
- `script/Deploy.s.sol` - deployment script for `HackathonEscrow`

## Notes

- ETH only; no ERC20 support in the MVP
- no upgradeability
- winner payout is all-or-nothing for the single-pot hackathon model
- set `ENTRY_FEE_WEI` before running the deploy script
