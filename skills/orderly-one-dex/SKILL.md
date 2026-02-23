---
name: orderly-one-dex
description: Create and manage a custom DEX using Orderly One API - deployment, custom domains, graduation, and theming
---

# Orderly Network: Orderly One DEX

This skill covers creating and managing a custom DEX using the Orderly One API.

## When to Use

- Launching a custom perpetuals DEX
- White-labeling Orderly's infrastructure
- Managing DEX deployment and configuration
- Custom domain and branding setup

## Prerequisites

- Orderly account
- Understanding of DEX operations
- Wallet for admin operations

## Overview

Orderly One allows you to launch your own perpetual futures DEX with:

- Custom branding and theming
- Custom domain
- Fee revenue sharing
- Full trading infrastructure

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Your Custom DEX                           │
│  - Custom domain (dex.yourproject.com)                      │
│  - Custom branding (logo, colors, theme)                    │
│  - Custom fee structure                                      │
└─────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   Orderly One Platform                       │
│  - Hosting & deployment                                      │
│  - Orderbook & matching engine                              │
│  - Liquidity pool                                           │
│  - Risk management                                          │
└─────────────────────────────────────────────────────────────┘
```

## API Base URL

```
https://api.dex.orderly.network
```

## Authentication

Orderly One uses JWT tokens obtained via wallet signature:

### Get Nonce

```typescript
GET /api/auth/nonce?wallet_address={address}

// Response
{
  "nonce": "abc123...",
  "expires_at": 1699123456
}
```

### Sign and Verify

```typescript
// 1. Get nonce
const nonceResponse = await fetch(
  `https://api.dex.orderly.network/api/auth/nonce?wallet_address=${address}`
);
const { nonce } = await nonceResponse.json();

// 2. Sign message
const message = `Sign this message to authenticate with Orderly One.\n\nNonce: ${nonce}`;
const signature = await wallet.signMessage(message);

// 3. Verify and get JWT
const verifyResponse = await fetch('https://api.dex.orderly.network/api/auth/verify', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    wallet_address: address,
    signature: signature,
    nonce: nonce,
  }),
});

const { token } = await verifyResponse.json();
// Use this token in Authorization: Bearer {token} header
```

---

## DEX Management

### Create DEX

```typescript
POST /api/dex
Authorization: Bearer {jwt_token}
Body: {
  name: "My Perps DEX",
  subdomain: "mydex",          // mydex.dex.orderly.network
  description: "Description",
  logo_url: "https://...",
  theme: {
    primary_color: "#6366f1",
    secondary_color: "#818cf8",
    background_color: "#0f172a",
  },
  trading_pairs: ["PERP_ETH_USDC", "PERP_BTC_USDC"],
  default_leverage: 10,
}

// Response
{
  "id": "dex_123",
  "name": "My Perps DEX",
  "subdomain": "mydex",
  "status": "deploying",
  "created_at": "2024-01-15T10:00:00Z"
}
```

### Get DEX

```typescript
GET /api/dex
Authorization: Bearer {jwt_token}

// Response
{
  "id": "dex_123",
  "name": "My Perps DEX",
  "subdomain": "mydex",
  "status": "active",
  "domain": "mydex.dex.orderly.network",
  "custom_domain": "dex.myproject.com",
  "trading_pairs": ["PERP_ETH_USDC", "PERP_BTC_USDC"],
  "theme": {...},
  "fees": {
    "maker_fee": 0.0002,
    "taker_fee": 0.0005,
  },
  "stats": {
    "daily_volume": "1000000",
    "active_users": 500,
    "total_trades": 50000,
  }
}
```

### Update DEX

```typescript
PUT /api/dex/{id}
Authorization: Bearer {jwt_token}
Body: {
  name: "Updated DEX Name",
  description: "New description",
  theme: {
    primary_color: "#10b981",
    secondary_color: "#34d399",
    background_color: "#1e293b",
  },
  trading_pairs: [
    "PERP_ETH_USDC",
    "PERP_BTC_USDC",
    "PERP_SOL_USDC"
  ],
}

// Response
{
  "success": true,
  "message": "DEX updated successfully"
}
```

### Delete DEX

```typescript
DELETE /api/dex/{id}
Authorization: Bearer {jwt_token}
Body: {
  confirmation: "DELETE_MY_DEX"  // Required confirmation string
}

// Response
{
  "success": true,
  "message": "DEX deletion initiated"
}
```

---

## Custom Domain

### Set Custom Domain

```typescript
POST /api/dex/{id}/custom-domain
Authorization: Bearer {jwt_token}
Body: {
  domain: "dex.myproject.com"
}

// Response
{
  "success": true,
  "dns_records": [
    {
      "type": "CNAME",
      "name": "dex",
      "value": "mydex.dex.orderly.network",
      "ttl": 3600
    },
    {
      "type": "TXT",
      "name": "_dex-verification",
      "value": "verify_abc123",
      "ttl": 3600
    }
  ],
  "status": "pending_verification"
}
```

### Verify Domain

After adding DNS records:

```typescript
GET /api/dex/{id}/custom-domain/verify
Authorization: Bearer {jwt_token}

// Response
{
  "verified": true,
  "ssl_status": "active",
  "domain": "dex.myproject.com"
}
```

### Remove Custom Domain

```typescript
DELETE /api/dex/{id}/custom-domain
Authorization: Bearer {jwt_token}
```

---

## Theme & Branding

### Update Social Card

```typescript
PUT /api/dex/social-card
Authorization: Bearer {jwt_token}
Body: {
  title: "My Perps DEX - Trade Perpetuals",
  description: "Trade BTC, ETH perpetuals with up to 50x leverage",
  image_url: "https://.../og-image.png",
  twitter_handle: "@myproject",
}
```

### AI Theme Generation

```typescript
POST /api/theme/generate
Authorization: Bearer {jwt_token}
Body: {
  prompt: "A futuristic dark theme with neon blue accents",
  base_style: "dark",  // "dark" | "light"
}

// Response
{
  "theme": {
    "primary_color": "#00d4ff",
    "secondary_color": "#7c3aed",
    "background_color": "#0a0a0f",
    "text_color": "#ffffff",
    "accent_color": "#00ff88",
    "border_color": "#1e293b",
    "success_color": "#10b981",
    "error_color": "#ef4444",
    "warning_color": "#f59e0b",
  },
  "css_vars": "--primary: #00d4ff; ...",
}
```

---

## Graduation (Demo → Full DEX)

Graduate from demo to full DEX with fee sharing:

### Check Graduation Status

```typescript
GET /api/graduation/status
Authorization: Bearer {jwt_token}

// Response
{
  "status": "not_graduated",  // "not_graduated" | "pending" | "graduated"
  "requirements": {
    "min_volume": "1000000",
    "current_volume": "500000",
    "min_users": 100,
    "current_users": 75,
  },
  "fee_options": [...],
}
```

### Get Fee Options

```typescript
GET /api/graduation/fee-options
Authorization: Bearer {jwt_token}

// Response
{
  "options": [
    {
      "id": "tier_1",
      "name": "Basic",
      "fee": "5000",      // USDC
      "fee_split": 0.5,   // 50% revenue share
      "features": ["Custom domain", "Basic analytics"]
    },
    {
      "id": "tier_2",
      "name": "Pro",
      "fee": "25000",
      "fee_split": 0.7,
      "features": ["Custom domain", "Advanced analytics", "Priority support"]
    },
    {
      "id": "tier_3",
      "name": "Enterprise",
      "fee": "100000",
      "fee_split": 0.85,
      "features": ["Custom domain", "Full analytics", "Dedicated support", "Custom features"]
    }
  ]
}
```

### Pay Graduation Fee

```typescript
// 1. Initiate fee payment
POST /api/graduation/pay
Authorization: Bearer {jwt_token}
Body: {
  tier: "tier_2",
  payment_method: "wallet",  // "wallet" | "orderly_balance"
}

// Response
{
  "payment_address": "0x...",
  "amount": "25000",
  "token": "USDC",
  "chain_id": 42161,
  "expires_at": "2024-01-16T10:00:00Z"
}

// 2. Send payment to address

// 3. Verify transaction
POST /api/graduation/verify-tx
Authorization: Bearer {jwt_token}
Body: {
  tx_hash: "0x...",
  chain_id: 42161,
}

// Response
{
  "verified": true,
  "broker_id": "my_dex_broker",
  "status": "graduated"
}
```

### Get Graduation Status After Payment

```typescript
GET /api/graduation/graduation-status
Authorization: Bearer {jwt_token}

// Response
{
  "graduated": true,
  "broker_id": "my_dex_broker",
  "tier": "tier_2",
  "fee_split": 0.7,
  "graduated_at": "2024-01-15T10:00:00Z"
}
```

---

## Leaderboard

### Get DEX Leaderboard

```typescript
GET /api/leaderboard
Query params:
  - period: "24h" | "7d" | "30d" | "all"
  - sort: "volume" | "users" | "trades"
  - limit: number

// Response
{
  "rankings": [
    {
      "rank": 1,
      "dex_id": "dex_123",
      "name": "Top DEX",
      "volume": "10000000",
      "users": 5000,
      "trades": 100000,
      "change_24h": "+15.5%",
    },
    // ...
  ],
  "my_dex": {
    "rank": 25,
    // ...
  }
}
```

### Set Leaderboard Visibility

```typescript
POST /api/dex/{id}/board-visibility
Authorization: Bearer {jwt_token}
Body: {
  visible: true
}
```

---

## Statistics

### Get DEX Stats

```typescript
GET /api/dex/{id}/stats
Authorization: Bearer {jwt_token}

// Response
{
  "volume": {
    "24h": "1000000",
    "7d": "7000000",
    "30d": "30000000",
    "all_time": "100000000",
  },
  "users": {
    "active_24h": 500,
    "active_7d": 2000,
    "total": 10000,
  },
  "trades": {
    "24h": 5000,
    "7d": 35000,
    "total": 500000,
  },
  "fees": {
    "24h": "500",
    "7d": "3500",
    "total": "50000",
  },
  "open_interest": "5000000",
}
```

---

## Network Selection

### Get Available Networks

```typescript
GET /api/dex/networks
Authorization: Bearer {jwt_token}

// Response
{
  "networks": [
    { "id": "arbitrum", "name": "Arbitrum", "chain_id": 42161 },
    { "id": "optimism", "name": "Optimism", "chain_id": 10 },
    { "id": "base", "name": "Base", "chain_id": 8453 },
    { "id": "ethereum", "name": "Ethereum", "chain_id": 1 },
    { "id": "solana", "name": "Solana", "chain_id": 900900900 },
  ]
}
```

---

## Rate Limiting

```typescript
GET /api/dex/rate-limit-status
Authorization: Bearer {jwt_token}

// Response
{
  "requests_remaining": 95,
  "requests_limit": 100,
  "reset_at": "2024-01-15T11:00:00Z"
}
```

---

## Deployment

### Check Workflow Status

```typescript
GET /api/dex/{id}/workflow-status?workflow=deploy
Authorization: Bearer {jwt_token}

// Response
{
  "status": "running",  // "pending" | "running" | "success" | "failed"
  "started_at": "2024-01-15T10:00:00Z",
  "steps": [
    { "name": "build", "status": "success" },
    { "name": "deploy", "status": "running" },
    { "name": "verify", "status": "pending" },
  ]
}
```

### Upgrade DEX

When new template versions are available:

```typescript
// Check for updates
GET /api/dex/{id}/upgrade-status
Authorization: Bearer {jwt_token}

// Response
{
  "update_available": true,
  "current_version": "1.0.0",
  "latest_version": "1.1.0",
  "changelog": "Bug fixes and performance improvements"
}

// Trigger upgrade
POST /api/dex/{id}/upgrade
Authorization: Bearer {jwt_token}

// Response
{
  "status": "upgrading",
  "estimated_time": "5 minutes"
}
```

---

## Common Issues

### DEX stuck in "deploying" status

- Check workflow status for error details
- Contact support if issue persists > 30 minutes

### Custom domain not working

- Verify DNS records are correctly configured
- Wait for DNS propagation (up to 48 hours)
- Check SSL certificate status

### Graduation payment not verified

- Ensure correct amount and token (USDC)
- Verify transaction on correct chain
- Allow time for confirmation

### Theme not applying

- Clear browser cache
- Check CSS variable syntax
- Verify theme object structure

## Related Skills

- **orderly-getting-started** - Account setup
- **orderly-trading-orders** - Trading functionality
- **orderly-ui-components** - UI components
