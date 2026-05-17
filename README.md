# StarPass Contracts

Soroban smart contracts for StarPass — a creator membership platform on Stellar where fans mint on-chain access passes to exclusive content tiers.

[![CI](https://github.com/laugh-tales/starpass-contracts/actions/workflows/ci.yml/badge.svg)](https://github.com/laugh-tales/starpass-contracts/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## What It Does

StarPass enables creators to define tiered memberships (Bronze, Silver, Gold) on-chain. Fans purchase access passes for a tier by paying USDC. The contract handles fee splitting, pass expiry, and creator withdrawals — all trustlessly on Stellar Soroban.

**How it works:**
1. A creator registers and defines membership tiers with a name, USDC price, and duration
2. A fan mints a pass for a tier — USDC is transferred, a Pass record is created on-chain
3. Any app or contract can call `has_valid_pass()` to gate access to content
4. The creator withdraws their earnings at any time

## Contract Functions

| Function | Who Calls It | Description |
|---|---|---|
| `initialize(admin, token, fee_bps)` | Deployer | Set up contract with USDC token and protocol fee |
| `register_creator(creator)` | Creator | Register as a creator on StarPass |
| `create_tier(creator, name, price, duration, max_supply)` | Creator | Define a new membership tier |
| `deactivate_tier(creator, tier_id)` | Creator | Stop new pass sales for a tier |
| `update_tier_price(creator, tier_id, new_price)` | Creator | Change tier price for future purchases |
| `mint_pass(fan, tier_id)` | Fan | Buy a pass — pays USDC, receives access |
| `withdraw(creator)` | Creator | Withdraw accumulated earnings |
| `has_valid_pass(fan, tier_id)` | Anyone | Check if fan has active non-expired pass |
| `get_pass(pass_id)` | Anyone | Get pass details |
| `get_tier(tier_id)` | Anyone | Get tier details |
| `get_creator(creator)` | Anyone | Get creator profile |
| `get_creator_balance(creator)` | Anyone | Get creator's pending withdrawal balance |
| `get_fan_passes(fan)` | Anyone | Get all pass IDs owned by a fan |
| `get_creator_tiers(creator)` | Anyone | Get all tier IDs for a creator |

## Pass Lifecycle
