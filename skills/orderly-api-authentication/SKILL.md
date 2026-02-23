---
name: orderly-api-authentication
description: Implement Ed25519 signature authentication for REST API requests, including header construction and request signing
---

# Orderly Network: API Authentication

This skill covers authentication for Orderly Network REST API using Ed25519 signatures.

## When to Use

- Building server-side trading bots
- Implementing direct API calls
- Understanding authentication flow
- Debugging signature issues

## Prerequisites

- Ed25519 key pair generated
- Account ID from registration
- Understanding of cryptographic signing

## Authentication Overview

Orderly uses Ed25519 elliptic curve signatures for API authentication:

```
1. Build message: timestamp + method + path + body
2. Sign message with Ed25519 private key
3. Encode signature as base64url
4. Include in request headers
```

## Required Headers

| Header               | Description                              |
| -------------------- | ---------------------------------------- |
| `orderly-timestamp`  | Unix timestamp in milliseconds           |
| `orderly-account-id` | Your Orderly account ID                  |
| `orderly-key`        | Your public key prefixed with `ed25519:` |
| `orderly-signature`  | Base64url-encoded Ed25519 signature      |

## Generating Ed25519 Key Pair

```typescript
import { getPublicKeyAsync, utils } from '@noble/ed25519';

// Generate private key (32 cryptographically secure random bytes)
const privateKey = utils.randomPrivateKey();

// Derive public key
const publicKey = await getPublicKeyAsync(privateKey);

// Encode public key as base58 (required by Orderly)
function encodeBase58(bytes: Uint8Array): string {
  const ALPHABET = '123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz';
  const BASE = 58n;

  let num = 0n;
  for (const byte of bytes) {
    num = num * 256n + BigInt(byte);
  }

  let result = '';
  while (num > 0n) {
    result = ALPHABET[Number(num % BASE)] + result;
    num = num / BASE;
  }

  // Handle leading zeros
  for (const byte of bytes) {
    if (byte === 0) {
      result = '1' + result;
    } else {
      break;
    }
  }

  return result;
}

const orderlyKey = `ed25519:${encodeBase58(publicKey)}`;

// Convert bytes to hex string (browser & Node.js compatible)
function bytesToHex(bytes: Uint8Array): string {
  return Array.from(bytes)
    .map((b) => b.toString(16).padStart(2, '0'))
    .join('');
}

console.log('Private Key (hex):', bytesToHex(privateKey));
console.log('Public Key (base58):', orderlyKey);

// STORE PRIVATE KEY SECURELY - NEVER SHARE IT
```

## Signing Requests

### Message Construction

```typescript
function buildSignMessage(timestamp: number, method: string, path: string, body?: string): string {
  // Message format: timestamp + method + path + body
  // Note: No spaces or separators between parts
  return `${timestamp}${method}${path}${body || ''}`;
}

// Examples
const timestamp = Date.now();

// GET request (no body)
const getMessage = buildSignMessage(timestamp, 'GET', '/v1/positions');

// POST request (with body)
const body = JSON.stringify({
  symbol: 'PERP_ETH_USDC',
  side: 'BUY',
  order_type: 'LIMIT',
  order_price: '3000',
  order_quantity: '0.1',
});
const postMessage = buildSignMessage(timestamp, 'POST', '/v1/order', body);
```

### Creating the Signature

```typescript
import { signAsync } from '@noble/ed25519';

async function signRequest(
  timestamp: number,
  method: string,
  path: string,
  body: string | undefined,
  privateKey: Uint8Array
): Promise<string> {
  const message = buildSignMessage(timestamp, method, path, body);

  // Sign with Ed25519
  const signatureBytes = await signAsync(new TextEncoder().encode(message), privateKey);

  // Encode as base64url (NOT base64)
  // Convert to base64, then make it URL-safe
  const base64 = btoa(String.fromCharCode(...signatureBytes));
  return base64.replace(/\+/g, '-').replace(/\//g, '_').replace(/=+$/, '');
}
```

## Sign and Send Request Helper

For a simple, standalone authentication helper that always works correctly with query parameters and proper Content-Type headers:

```typescript
import { getPublicKeyAsync, signAsync } from '@noble/ed25519';
import { encodeBase58 } from 'ethers';

export async function signAndSendRequest(
  orderlyAccountId: string,
  privateKey: Uint8Array | string,
  input: URL | string,
  init?: RequestInit | undefined
): Promise<Response> {
  const timestamp = Date.now();
  const encoder = new TextEncoder();

  const url = new URL(input);
  let message = `${String(timestamp)}${init?.method ?? 'GET'}${url.pathname}${url.search}`;
  if (init?.body) {
    message += init.body;
  }
  const orderlySignature = await signAsync(encoder.encode(message), privateKey);

  return fetch(input, {
    headers: {
      'Content-Type':
        init?.method !== 'GET' && init?.method !== 'DELETE'
          ? 'application/json'
          : 'application/x-www-form-urlencoded',
      'orderly-timestamp': String(timestamp),
      'orderly-account-id': orderlyAccountId,
      'orderly-key': `ed25519:${encodeBase58(await getPublicKeyAsync(privateKey))}`,
      'orderly-signature': Buffer.from(orderlySignature).toString('base64url'),
      ...(init?.headers ?? {}),
    },
    ...(init ?? {}),
  });
}
```

This helper function:

- Properly parses the URL to extract both pathname and search (query) parameters
- Correctly sets Content-Type based on HTTP method (GET/DELETE use `application/x-www-form-urlencoded`, others use `application/json`)
- Constructs the signature message with timestamp + method + pathname + search + body
- Returns the fetch response for further processing

### Usage Examples

```typescript
const baseUrl = 'https://api.orderly.org';
const accountId = '0x123...';
const privateKey = new Uint8Array(32); // Your private key

// GET request with query parameters
const positions = await signAndSendRequest(accountId, privateKey, `${baseUrl}/v1/positions`);
const positionsData = await positions.json();

// GET request with query params
const orders = await signAndSendRequest(
  accountId,
  privateKey,
  `${baseUrl}/v1/orders?symbol=PERP_ETH_USDC&status=INCOMPLETE`
);
const ordersData = await orders.json();

// POST request with body
const order = await signAndSendRequest(accountId, privateKey, `${baseUrl}/v1/order`, {
  method: 'POST',
  body: JSON.stringify({
    symbol: 'PERP_ETH_USDC',
    side: 'BUY',
    order_type: 'LIMIT',
    order_price: '3000',
    order_quantity: '0.1',
  }),
});
const orderData = await order.json();

// DELETE request
const cancel = await signAndSendRequest(
  accountId,
  privateKey,
  `${baseUrl}/v1/order?order_id=123&symbol=PERP_ETH_USDC`,
  { method: 'DELETE' }
);
```

### Error Handling Helper

```typescript
class OrderlyApiError extends Error {
  code: number;
  details: any;

  constructor(response: any) {
    super(response.message || 'API Error');
    this.code = response.code;
    this.details = response;
  }
}

// Usage with error handling
async function apiRequest(
  accountId: string,
  privateKey: Uint8Array,
  url: string,
  init?: RequestInit
) {
  const response = await signAndSendRequest(accountId, privateKey, url, init);
  const result = await response.json();

  if (!result.success) {
    throw new OrderlyApiError(result);
  }

  return result.data;
}
```

## Query Parameters

Query parameters must be included in the signature message. The URL is parsed to extract both pathname and search parameters:

```typescript
// Correct - query params are parsed from the URL
const url = new URL('/v1/orders?symbol=PERP_ETH_USDC&status=INCOMPLETE', baseUrl);
// Message: timestamp + method + pathname + search
// Result: "1234567890123GET/v1/orders?symbol=PERP_ETH_USDC&status=INCOMPLETE"

// Wrong - query params added separately after signing
const path = '/v1/orders';
const signature = await sign(timestamp, 'GET', path);
const url = `${path}?symbol=PERP_ETH_USDC`; // Signature mismatch!
```

## Common Errors

### Signature Mismatch (Code 10016)

```
Cause: Signature doesn't match expected value

Check:
1. Message format: timestamp + method + path + body (no spaces)
2. Method is uppercase: GET, POST, DELETE, PUT
3. Path includes query parameters
4. Body is exact JSON string (same whitespace)
5. Signature is base64url encoded (not base64)
```

### Timestamp Expired (Code 10017)

```
Cause: Timestamp is too old or too far in the future

Solution:
- Ensure server clock is synchronized
- Timestamp must be within ±30 seconds
- Generate timestamp immediately before signing
```

### Invalid Orderly Key (Code 10019)

```
Cause: Public key format incorrect

Solution:
- Must be prefixed with 'ed25519:'
- Public key must be base58 encoded
- Key must be registered to account
```

## Orderly Key Scopes

When registering an API key, specify permissions:

| Scope     | Permissions                          |
| --------- | ------------------------------------ |
| `read`    | Read positions, orders, balance      |
| `trading` | Place, cancel, modify orders         |
| `asset`   | Deposit, withdraw, internal transfer |

```typescript
// When adding key via EIP-712 signing
const addKeyMessage = {
  brokerId: 'woofi_dex',
  chainId: 42161,
  orderlyKey: 'ed25519:...',
  scope: 'read,trading', // Multiple scopes comma-separated
  timestamp: Date.now(),
  expiration: Date.now() + 31536000000, // 1 year
};
```

## Security Best Practices

### Store Private Keys Securely

```typescript
// NEVER hardcode private keys
// BAD:
const privateKey = new Uint8Array([1, 2, 3, ...]);

// GOOD: Load from environment
const privateKeyHex = process.env.ORDERLY_PRIVATE_KEY;
// Convert hex string to Uint8Array (browser & Node.js compatible)
function hexToBytes(hex: string): Uint8Array {
  const bytes = new Uint8Array(hex.length / 2);
  for (let i = 0; i < hex.length; i += 2) {
    bytes[i / 2] = parseInt(hex.slice(i, i + 2), 16);
  }
  return bytes;
}
const privateKey = hexToBytes(privateKeyHex);

// BETTER: Use secure key management
// AWS KMS, HashiCorp Vault, etc.
```

### Key Rotation

Rotate your API keys periodically for security:

```typescript
// Generate new key pair
const newPrivateKey = utils.randomPrivateKey();
const newPublicKey = await getPublicKeyAsync(newPrivateKey);

// Register new key (requires wallet signature via EIP-712)
// POST /v1/orderly_key - No Ed25519 auth required
const orderlyKey = `ed25519:${encodeBase58(newPublicKey)}`;
const timestamp = Date.now();
const expiration = timestamp + 31536000000; // 1 year

const addKeyMessage = {
  brokerId: 'your_broker_id',
  chainId: 42161, // Arbitrum mainnet
  orderlyKey: orderlyKey,
  scope: 'read,trading', // Comma-separated scopes
  timestamp: timestamp,
  expiration: expiration,
};

// Sign with wallet (EIP-712)
const addKeySignature = await wallet.signTypedData({
  domain: {
    name: 'Orderly',
    version: '1',
    chainId: 42161,
    verifyingContract: '0x...', // Contract address
  },
  types: {
    EIP712Domain: [
      { name: 'name', type: 'string' },
      { name: 'version', type: 'string' },
      { name: 'chainId', type: 'uint256' },
      { name: 'verifyingContract', type: 'address' },
    ],
    AddOrderlyKey: [
      { name: 'brokerId', type: 'string' },
      { name: 'chainId', type: 'uint256' },
      { name: 'orderlyKey', type: 'string' },
      { name: 'scope', type: 'string' },
      { name: 'timestamp', type: 'uint256' },
      { name: 'expiration', type: 'uint256' },
    ],
  },
  primaryType: 'AddOrderlyKey',
  message: addKeyMessage,
});

const registerResponse = await fetch('https://api.orderly.org/v1/orderly_key', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    message: addKeyMessage,
    signature: addKeySignature,
    userAddress: walletAddress,
  }),
});

const registerResult = await registerResponse.json();
if (!registerResult.success) {
  throw new Error(`Failed to register key: ${registerResult.message}`);
}

// Update your application config
config.privateKey = newPrivateKey;
config.orderlyKey = orderlyKey;

// Remove old key using authenticated request
// POST /v1/client/remove_orderly_key - Requires Ed25519 auth
const oldOrderlyKey = `ed25519:${encodeBase58(oldPublicKey)}`;
const removeResponse = await signAndSendRequest(
  accountId,
  newPrivateKey, // Use the NEW key to authenticate
  'https://api.orderly.org/v1/client/remove_orderly_key',
  {
    method: 'POST',
    body: JSON.stringify({
      orderly_key: oldOrderlyKey,
    }),
  }
);

const removeResult = await removeResponse.json();
if (!removeResult.success) {
  throw new Error(`Failed to remove old key: ${removeResult.message}`);
}
```

### IP Restrictions

```typescript
// Set IP restriction for key
POST /v1/client/set_orderly_key_ip_restriction
Body: {
  orderly_key: 'ed25519:...',
  ip_list: ['1.2.3.4', '5.6.7.8'],
}

// Get current restrictions
GET /v1/client/orderly_key_ip_restriction?orderly_key={key}

// Reset (remove) restrictions
POST /v1/client/reset_orderly_key_ip_restriction
Body: { orderly_key: 'ed25519:...' }
```

## WebSocket Authentication

WebSocket also requires Ed25519 authentication:

```typescript
const ws = new WebSocket(`wss://ws-private-evm.orderly.org/v2/ws/private/stream/${accountId}`);

ws.onopen = async () => {
  const timestamp = Date.now();
  const message = timestamp.toString();

  const signature = await signAsync(new TextEncoder().encode(message), privateKey);

  // Convert to base64url (browser & Node.js compatible)
  const base64 = btoa(String.fromCharCode(...signature));
  const base64url = base64.replace(/\+/g, '-').replace(/\//g, '_').replace(/=+$/, '');

  ws.send(
    JSON.stringify({
      id: 'auth_1',
      event: 'auth',
      params: {
        orderly_key: orderlyKey,
        sign: base64url,
        timestamp: timestamp,
      },
    })
  );
};
```

## Testing Authentication

```typescript
// Verify key is valid
GET /v1/get_orderly_key?orderly_key={key}

// Response
{
  "success": true,
  "data": {
    "account_id": "0x...",
    "valid": true,
    "scope": "read,trading",
    "expires_at": 1735689600000
  }
}
```

## Related Skills

- **orderly-getting-started** - Account and key registration
- **orderly-trading-orders** - Using authenticated endpoints
- **orderly-websocket-streaming** - WebSocket authentication
