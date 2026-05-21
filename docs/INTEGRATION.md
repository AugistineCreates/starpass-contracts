# Integration Guide

This guide explains how to integrate StarPass into your Stellar application. Any app or contract can call `has_valid_pass()` to gate access to content behind a membership check.

## Overview

StarPass is a creator membership platform where fans mint on-chain access passes. The contract exposes a simple access-check function that any application can use:

```
has_valid_pass(fan_address, tier_id) → true | false
```

If this returns `true`, the fan has a valid, non-expired pass for that tier. Your app can use this to show or hide content, unlock features, or gate API responses.

## JavaScript / TypeScript Integration

Install the Stellar SDK:

```bash
npm install @stellar/stellar-sdk
```

Check if a fan has a valid pass:

```typescript
import * as StellarSdk from '@stellar/stellar-sdk';

const RPC_URL = 'https://soroban-testnet.stellar.org';
const CONTRACT_ID = 'YOUR_STARPASS_CONTRACT_ID';
const NETWORK = StellarSdk.Networks.TESTNET;

async function hasValidPass(
  fanAddress: string,
  tierId: number
): Promise<boolean> {
  const server = new StellarSdk.rpc.Server(RPC_URL);
  const contract = new StellarSdk.Contract(CONTRACT_ID);

  const account = await server.getAccount(fanAddress);

  const tx = new StellarSdk.TransactionBuilder(account, {
    fee: '100',
    networkPassphrase: NETWORK,
  })
    .addOperation(
      contract.call(
        'has_valid_pass',
        StellarSdk.nativeToScVal(fanAddress, { type: 'address' }),
        StellarSdk.nativeToScVal(tierId, { type: 'u32' }),
      ),
    )
    .setTimeout(30)
    .build();

  const result = await server.simulateTransaction(tx);

  if ('error' in result) return false;

  return StellarSdk.scValToNative(result.result?.retval) as boolean;
}

// Usage
const isSubscriber = await hasValidPass('GFAN...', 1);
if (isSubscriber) {
  // show premium content
}
```

## Checking Any Pass from a Creator

To check if a fan has **any** valid pass from a specific creator (regardless of tier):

```typescript
async function hasAnyValidPass(
  fanAddress: string,
  creatorAddress: string
): Promise<boolean> {
  // 1. Get all tier IDs for the creator
  const tiers = await getCreatorTiers(creatorAddress);

  // 2. Check each tier
  for (const tierId of tiers) {
    const valid = await hasValidPass(fanAddress, tierId);
    if (valid) return true;
  }

  return false;
}

async function getCreatorTiers(creatorAddress: string): Promise<number[]> {
  const server = new StellarSdk.rpc.Server(RPC_URL);
  const contract = new StellarSdk.Contract(CONTRACT_ID);
  const account = await server.getAccount(creatorAddress);

  const tx = new StellarSdk.TransactionBuilder(account, {
    fee: '100',
    networkPassphrase: NETWORK,
  })
    .addOperation(
      contract.call(
        'get_creator_tiers',
        StellarSdk.nativeToScVal(creatorAddress, { type: 'address' }),
      ),
    )
    .setTimeout(30)
    .build();

  const result = await server.simulateTransaction(tx);
  if ('error' in result) return [];

  return StellarSdk.scValToNative(result.result?.retval) as number[];
}
```

## Backend API Middleware (NestJS / Express)

Gate an API endpoint so only valid pass holders can access it:

```typescript
// starpass.guard.ts
import { Injectable, CanActivate, ExecutionContext, ForbiddenException } from '@nestjs/common';
import { hasValidPass } from './starpass.service';

@Injectable()
export class StarPassGuard implements CanActivate {
  constructor(private readonly tierId: number) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const fanAddress = request.user?.stellarAddress;

    if (!fanAddress) throw new ForbiddenException('Not authenticated');

    const valid = await hasValidPass(fanAddress, this.tierId);
    if (!valid) throw new ForbiddenException('No valid pass for this tier');

    return true;
  }
}

// Usage in a controller
@Get('premium-content')
@UseGuards(new StarPassGuard(1)) // tier ID 1
getPremiumContent() {
  return { content: 'This is exclusive content for tier 1 subscribers' };
}
```

## Contract-to-Contract Integration

To call `has_valid_pass` from another Soroban contract:

```rust
use soroban_sdk::{contractclient, Address, Env};

#[contractclient(name = "StarPassClient")]
pub trait StarPassInterface {
    fn has_valid_pass(env: Env, fan: Address, tier_id: u32) -> bool;
}

// In your contract:
pub fn gated_action(env: Env, caller: Address, starpass_contract: Address) {
    caller.require_auth();

    let starpass = StarPassClient::new(&env, &starpass_contract);
    let has_pass = starpass.has_valid_pass(&caller, &1u32);

    assert!(has_pass, "Valid StarPass required");

    // proceed with gated logic
}
```

## Testnet Contract Address

The StarPass contract is deployed on Stellar testnet. Update this after deployment:

```
CONTRACT_ID=REPLACE_WITH_DEPLOYED_CONTRACT_ID
NETWORK=testnet
RPC_URL=https://soroban-testnet.stellar.org
```

See [DEPLOYMENT.md](./DEPLOYMENT.md) for full deployment instructions.

## Pass Lifecycle Reference

| State | `has_valid_pass` returns |
|---|---|
| Pass minted, not expired | `true` |
| Pass expired | `false` |
| No pass ever minted | `false` |
| Tier deactivated | `false` (existing passes still valid until expiry) |

## Error Handling

| Scenario | Behaviour |
|---|---|
| Fan address not found on-chain | Returns `false` |
| Invalid contract ID | Simulation throws — handle with try/catch |
| RPC timeout | Simulation throws — retry with backoff |
| Tier ID does not exist | Contract panics — simulation returns error |
