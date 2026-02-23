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
import { getPublicKeyAsync } from '@noble/ed25519';
import { randomBytes } from 'crypto';

// Generate private key (32 bytes)
const privateKey = randomBytes(32);

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

console.log('Private Key (hex):', Buffer.from(privateKey).toString('hex'));
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
  return Buffer.from(signatureBytes).toString('base64url');
}
```

## Complete API Client

```typescript
import { signAsync } from '@noble/ed25519';

interface OrderlyConfig {
  accountId: string;
  orderlyKey: string; // ed25519:{base58PublicKey}
  privateKey: Uint8Array;
  baseUrl: string; // 'https://api.orderly.org' or 'https://testnet-api.orderly.org'
}

class OrderlyClient {
  private config: OrderlyConfig;

  constructor(config: OrderlyConfig) {
    this.config = config;
  }

  private async sign(
    timestamp: number,
    method: string,
    path: string,
    body?: string
  ): Promise<string> {
    const message = `${timestamp}${method}${path}${body || ''}`;
    const signature = await signAsync(new TextEncoder().encode(message), this.config.privateKey);
    return Buffer.from(signature).toString('base64url');
  }

  private async request<T>(method: string, path: string, body?: any): Promise<T> {
    const timestamp = Date.now();
    const bodyStr = body ? JSON.stringify(body) : undefined;
    const signature = await this.sign(timestamp, method, path, bodyStr);

    const headers: Record<string, string> = {
      'Content-Type': 'application/json',
      'orderly-timestamp': String(timestamp),
      'orderly-account-id': this.config.accountId,
      'orderly-key': this.config.orderlyKey,
      'orderly-signature': signature,
    };

    const options: RequestInit = {
      method,
      headers,
    };

    if (bodyStr) {
      options.body = bodyStr;
    }

    const response = await fetch(`${this.config.baseUrl}${path}`, options);
    const result = await response.json();

    if (!result.success) {
      throw new OrderlyApiError(result);
    }

    return result.data;
  }

  // Public endpoints (no auth required for some)
  async getMarkets() {
    return this.request('GET', '/v1/public/futures');
  }

  async getOrderbook(symbol: string) {
    return this.request('GET', `/v1/orderbook/${symbol}`);
  }

  // Private endpoints
  async getPositions() {
    return this.request('GET', '/v1/positions');
  }

  async getBalance() {
    return this.request('GET', '/v1/client/holding');
  }

  async getOrders(options?: { symbol?: string; status?: string }) {
    const params = new URLSearchParams();
    if (options?.symbol) params.set('symbol', options.symbol);
    if (options?.status) params.set('status', options.status);
    const query = params.toString() ? `?${params.toString()}` : '';
    return this.request('GET', `/v1/orders${query}`);
  }

  async placeOrder(order: {
    symbol: string;
    side: 'BUY' | 'SELL';
    order_type: string;
    order_price?: string;
    order_quantity: string;
  }) {
    return this.request('POST', '/v1/order', order);
  }

  async cancelOrder(orderId: string, symbol: string) {
    return this.request('DELETE', `/v1/order?order_id=${orderId}&symbol=${symbol}`);
  }

  async cancelAllOrders(symbol?: string) {
    const query = symbol ? `?symbol=${symbol}` : '';
    return this.request('DELETE', `/v1/orders${query}`);
  }

  async setLeverage(symbol: string, leverage: number) {
    return this.request('POST', '/v1/client/leverage', { symbol, leverage });
  }
}

class OrderlyApiError extends Error {
  code: number;
  details: any;

  constructor(response: any) {
    super(response.message || 'API Error');
    this.code = response.code;
    this.details = response;
  }
}

// Usage
const client = new OrderlyClient({
  accountId: '0x123...',
  orderlyKey: 'ed25519:5Yw...',
  privateKey: new Uint8Array(32), // Your private key
  baseUrl: 'https://api.orderly.org',
});

// Get positions
const positions = await client.getPositions();

// Place order
const order = await client.placeOrder({
  symbol: 'PERP_ETH_USDC',
  side: 'BUY',
  order_type: 'LIMIT',
  order_price: '3000',
  order_quantity: '0.1',
});
```

## Query Parameters

For GET requests with query parameters, include them in the path:

```typescript
// Correct - include query params in path
const path = '/v1/orders?symbol=PERP_ETH_USDC&status=INCOMPLETE';
const signature = await sign(timestamp, 'GET', path);

// Wrong - query params not in path
const path = '/v1/orders';
// Query params added separately will cause signature mismatch
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
const privateKey = Buffer.from(privateKeyHex, 'hex');

// BETTER: Use secure key management
// AWS KMS, HashiCorp Vault, etc.
```

### Key Rotation

```typescript
// Generate new key pair
const newPrivateKey = crypto.getRandomValues(new Uint8Array(32));
const newPublicKey = await getPublicKeyAsync(newPrivateKey);

// Register new key (requires wallet signature)
await registerNewKey(newPublicKey);

// Update your application config
config.privateKey = newPrivateKey;
config.orderlyKey = `ed25519:${encodeBase58(newPublicKey)}`;

// Remove old key (optional)
await removeOldKey(oldPublicKey);
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

  ws.send(
    JSON.stringify({
      id: 'auth_1',
      event: 'auth',
      params: {
        orderly_key: orderlyKey,
        sign: Buffer.from(signature).toString('base64url'),
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
