---
name: orderly-getting-started
description: Complete onboarding workflow for Orderly Network - wallet connection, EIP-712 signing, Ed25519 key generation, account registration, and first deposit setup
---

# Orderly Network: Getting Started

This skill guides you through the complete onboarding process for Orderly Network - from wallet connection to placing your first trade.

## When to Use

- Setting up a new Orderly Network integration
- Registering a new trading account
- Generating API credentials for programmatic trading
- Understanding the onboarding flow

## Prerequisites

- Node.js 18+ installed
- A Web3 wallet (MetaMask, WalletConnect, etc.)
- A Broker ID (e.g., `woofi_dex`, or your own)
- Basic understanding of EIP-712 typed data signing

## Architecture Overview

Orderly Network is an omnichain perpetual futures trading infrastructure:

```
┌─────────────────────────────────────────────────────────────┐
│                     User's Wallet                            │
│  (MetaMask, WalletConnect, Solana wallets)                  │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   Orderly SDK / API                          │
│  - React Hooks (@orderly.network/hooks)                     │
│  - REST API (api.orderly.org)                               │
│  - WebSocket (ws.orderly.org)                               │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   Orderly Network                            │
│  - Orderbook Matching Engine                                │
│  - Risk Management System                                   │
│  - Cross-chain Vault                                        │
└─────────────────────────────────────────────────────────────┘
```

## Step 1: Check Account Registration

Before attempting registration, verify if the wallet already has an account:

```typescript
const BROKER_ID = 'woofi_dex'; // Your broker ID
const walletAddress = '0x...'; // User's wallet address

const response = await fetch(
  `https://testnet-api.orderly.org/v1/get_account?broker_id=${BROKER_ID}&user_address=${walletAddress}`
);

const data = await response.json();
// If data.success is true, account already exists
// If not, proceed with registration
```

## Step 2: Fetch Registration Nonce

Retrieve a unique nonce required for registration (valid for 2 minutes):

```typescript
const nonceResponse = await fetch('https://testnet-api.orderly.org/v1/registration_nonce');
const { data: nonce } = await nonceResponse.json();
console.log('Registration nonce:', nonce);
```

## Step 3: Sign Registration Message (EIP-712)

Create and sign an EIP-712 typed message:

```typescript
// EIP-712 Domain Configuration
const OFF_CHAIN_DOMAIN = {
  name: 'Orderly',
  version: '1',
  chainId: 421614, // Arbitrum Sepolia testnet (42161 for mainnet)
  verifyingContract: '0xCcCCccccCCCCcCCCCCCcCcCccCcCCCcCcccccccC',
};

// Registration Message Type
const MESSAGE_TYPES = {
  Registration: [
    { name: 'brokerId', type: 'string' },
    { name: 'chainId', type: 'uint256' },
    { name: 'timestamp', type: 'uint64' },
    { name: 'registrationNonce', type: 'uint256' },
  ],
};

// Create the message
const registerMessage = {
  brokerId: BROKER_ID,
  chainId: 421614,
  timestamp: Date.now(),
  registrationNonce: nonce,
};

// Sign with wallet (e.g., MetaMask)
const signature = await window.ethereum.request({
  method: 'eth_signTypedData',
  params: [
    walletAddress,
    {
      types: MESSAGE_TYPES,
      domain: OFF_CHAIN_DOMAIN,
      message: registerMessage,
      primaryType: 'Registration',
    },
  ],
});
```

## Step 4: Submit Registration

Send the signed payload to create the Orderly Account ID:

```typescript
const registerResponse = await fetch('https://testnet-api.orderly.org/v1/register_account', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    message: registerMessage,
    signature: signature,
    userAddress: walletAddress,
  }),
});

const result = await registerResponse.json();
console.log('Account ID:', result.data.account_id);
```

## Step 5: Generate Ed25519 Key Pair

Generate an Ed25519 key pair for API authentication:

```typescript
import { getPublicKeyAsync } from '@noble/ed25519';

// Generate 32-byte private key
const privateKey = crypto.getRandomValues(new Uint8Array(32));

// Derive public key
const publicKey = await getPublicKeyAsync(privateKey);

// Encode public key as base58 (required by Orderly)
function encodeBase58(bytes: Uint8Array): string {
  const ALPHABET = '123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz';
  let result = '';
  let num = 0n;
  for (const byte of bytes) {
    num = num * 256n + BigInt(byte);
  }
  while (num > 0n) {
    result = ALPHABET[Number(num % 58n)] + result;
    num = num / 58n;
  }
  return result;
}

const orderlyKey = `ed25519:${encodeBase58(publicKey)}`;
console.log('Orderly Key:', orderlyKey);
console.log('Secret Key (STORE SECURELY!):', Buffer.from(privateKey).toString('hex'));
```

## Step 6: Sign Add Orderly Key Message (EIP-712)

Associate the Ed25519 key with your account:

```typescript
const ADD_KEY_TYPES = {
  AddOrderlyKey: [
    { name: 'brokerId', type: 'string' },
    { name: 'chainId', type: 'uint256' },
    { name: 'orderlyKey', type: 'string' },
    { name: 'scope', type: 'string' },
    { name: 'timestamp', type: 'uint64' },
    { name: 'expiration', type: 'uint64' },
  ],
};

const addKeyMessage = {
  brokerId: BROKER_ID,
  chainId: 421614,
  orderlyKey: orderlyKey,
  scope: 'read,trading', // Permissions: read, trading, asset
  timestamp: Date.now(),
  expiration: Date.now() + 31536000000, // 1 year from now
};

const addKeySignature = await window.ethereum.request({
  method: 'eth_signTypedData',
  params: [
    walletAddress,
    {
      types: ADD_KEY_TYPES,
      domain: OFF_CHAIN_DOMAIN,
      message: addKeyMessage,
      primaryType: 'AddOrderlyKey',
    },
  ],
});
```

## Step 7: Submit Orderly Key

Register the API key for trading:

```typescript
const keyResponse = await fetch('https://testnet-api.orderly.org/v1/orderly_key', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    message: addKeyMessage,
    signature: addKeySignature,
    userAddress: walletAddress,
  }),
});

const keyResult = await keyResponse.json();
console.log('Key registered:', keyResult.success);
```

## Using the React SDK (Recommended)

For React applications, use the Orderly SDK which handles all of this automatically:

```typescript
import { OrderlyAppProvider, WalletConnector } from '@orderly.network/react';
import { useAccount } from '@orderly.network/hooks';

function App() {
  return (
    <OrderlyAppProvider
      brokerId="woofi_dex"
      chainFilter={[421614, 42161]} // Arbitrum testnet & mainnet
    >
      <WalletConnector />
      <TradingApp />
    </OrderlyAppProvider>
  );
}

function TradingApp() {
  const { account, state } = useAccount();

  if (state.status === 'notConnected') {
    return <button onClick={() => account.connect()}>Connect Wallet</button>;
  }

  if (state.status === 'connecting') {
    return <div>Connecting...</div>;
  }

  return <div>Account ID: {account.accountId}</div>;
}
```

## Quick Setup with SDK

```bash
# Install dependencies
npm install @orderly.network/hooks @orderly.network/react @orderly.network/types

# Or with yarn
yarn add @orderly.network/hooks @orderly.network/react @orderly.network/types
```

```typescript
// Minimal setup example
import { OrderlyAppProvider, WalletConnect } from '@orderly.network/react';

export function App() {
  return (
    <OrderlyAppProvider brokerId="woofi_dex">
      <WalletConnect />
    </OrderlyAppProvider>
  );
}
```

## Environment Configuration

| Environment | API Base URL                      | WebSocket URL                            | Chain ID             |
| ----------- | --------------------------------- | ---------------------------------------- | -------------------- |
| Mainnet     | `https://api.orderly.org`         | `wss://ws.orderly.org/ws/stream`         | 42161 (Arbitrum)     |
| Testnet     | `https://testnet-api.orderly.org` | `wss://testnet-ws.orderly.org/ws/stream` | 421614 (Arb Sepolia) |

## Supported Chains

| Chain    | Chain ID              | Mainnet | Testnet |
| -------- | --------------------- | ------- | ------- |
| Arbitrum | 42161 / 421614        | ✅      | ✅      |
| Optimism | 10 / 11155420         | ✅      | ✅      |
| Base     | 8453 / 84532          | ✅      | ✅      |
| Ethereum | 1 / 11155111          | ✅      | ✅      |
| Solana   | 900900900 / 901901901 | ✅      | ✅      |
| Mantle   | 5000 / 5003           | ✅      | ✅      |

## Common Issues

### "Nonce expired" error

- Nonces are valid for 2 minutes only
- Fetch a new nonce and retry

### "Account already exists" error

- The wallet is already registered with this broker
- Use `/v1/get_account` to retrieve existing account info

### "Invalid signature" error

- Ensure the EIP-712 domain matches exactly
- Check chain ID matches your network
- Verify the message structure matches the types

### Wallet connection fails

- Ensure you're on the correct network
- Check if the wallet supports EIP-712 signing
- Try refreshing the page and reconnecting

## Related Skills

- **orderly-api-authentication** - Deep dive into Ed25519 signing
- **orderly-deposit-withdraw** - Fund your account
- **orderly-trading-orders** - Place your first order
- **orderly-sdk-react-hooks** - Full SDK reference
