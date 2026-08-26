---
name: orderly-one-dex
description: Create and manage a custom white-label DEX using Orderly One - launch paths, deployment, custom domains, graduation with x402 payment, theming, and admin operations
---

# Orderly Network: Orderly One DEX

**Orderly One** ([https://dex.orderly.network](https://dex.orderly.network)) is a platform that lets anyone launch a white-label perpetual-futures DEX on Orderly Network — with or without code. It serves as both:

1. **A web UI** at [dex.orderly.network](https://dex.orderly.network) where humans can create, configure, and deploy a DEX through a step-by-step wizard.
2. **A REST API** that an AI agent or script can call to do the same thing programmatically.

> **Important:** For graduation and broker ID creation — the step that unlocks fee revenue — agents can pay via the x402 HTTP payment protocol (`POST /api/graduation/graduate`) and register a broker in a single request, no browser needed. The web portal at [https://dex.orderly.network/dex](https://dex.orderly.network/dex) remains the easiest path for non-technical users; always inform them this option exists.

## Two Launch Paths

| Path               | Description                                                                                                                          | Graduation Fee | Who It's For                                            |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------ | -------------- | ------------------------------------------------------- |
| **Low-code**       | Create a branded DEX frontend via Orderly One portal or API. It forks a template repo and deploys to GitHub Pages.                   | $100           | Teams wanting fast launch with minimal frontend work    |
| **Custom SDK/API** | Use the Orderly SDK or API to build a fully custom frontend. Graduate via Orderly One to get a broker ID — no DEX frontend required. | $10            | Wallets, existing exchanges, teams wanting full control |

See the official builder onboarding guide: [https://orderly.network/docs/introduction/getting-started/builder-onboarding](https://orderly.network/docs/introduction/getting-started/builder-onboarding)

## When to Use

- Creating a custom perpetuals DEX
- Managing DEX deployment, domains, or themes
- Handling graduation for fee sharing

---

## Key Concepts

- **Broker ID**: A unique identifier in the Orderly ecosystem. Starts as `"demo"` and becomes a real broker ID after graduation. This is how your DEX earns trading fees.
- **Graduation**: The process of converting a demo DEX into a fee-earning DEX by paying $10-$100 and registering as a broker.
- **Template Repository**: The GitHub repo that gets forked for each low-code DEX. Contains the Orderly SDK trading frontend.
- **One DEX Per User**: Each wallet address can create exactly one DEX.

---

## API Base URLs

| Environment | Base URL                                  |
| ----------- | ----------------------------------------- |
| Mainnet     | `https://dex-api.orderly.network`         |
| Testnet     | `https://testnet-dex-api.orderly.network` |

The API exposes an interactive OpenAPI spec (Scalar) at the root: `GET /`

## Graduation-Supported Chains (Payment)

Mainnet: Ethereum (1), Arbitrum (42161), Base (8453)
Testnet: Sepolia, Arbitrum Sepolia, Base Sepolia

---

## API Categories

Use `get_orderly_one_api_info` MCP tool for full endpoint details.

| Category        | Description                      | Key Endpoints                                                                                                           |
| --------------- | -------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **auth**        | Wallet signature authentication  | `/api/auth/nonce`, `/api/auth/verify`, `/api/auth/validate`                                                             |
| **dex**         | DEX CRUD, domains, deployment    | `/api/dex`, `/api/dex/{id}`, `/api/dex/{id}/custom-domain`, `/api/dex/{id}/workflow-status`                             |
| **theme**       | AI theme generation              | `/api/theme/modify`, `/api/theme/fine-tune`                                                                             |
| **graduation**  | Demo → full DEX with fee sharing | `/api/graduation/graduate` (x402), `/api/graduation/verify-tx`, `/api/graduation/fee-options`, `/api/graduation/refund` |
| **leaderboard** | Cross-DEX rankings               | `/api/leaderboard`, `/api/leaderboard/broker/{brokerId}`                                                                |
| **stats**       | Platform statistics              | `/api/stats`, `/api/stats/swap-fee-config`                                                                              |

---

## Create/Update DEX

Both `POST /api/dex` (create) and `PUT /api/dex/{id}` (update) use `multipart/form-data`.

### Required Fields

| Field        | Type   | Constraints                               |
| ------------ | ------ | ----------------------------------------- |
| `brokerName` | string | 3-30 chars, alphanumeric/space/dot/hyphen |

### Optional Fields

**Chains:**
| Field | Type | Notes |
| -------------- | ---------------- | ------------------------ |
| `chainIds` | number[] (JSON) | e.g. `[42161, 10, 8453]` |
| `defaultChain` | number | Default chain ID |

**Branding (files):**
| Field | Type | Max Size |
| --------------- | ---- | -------- |
| `primaryLogo` | File | 250KB |
| `secondaryLogo` | File | 100KB |
| `favicon` | File | 50KB |
| `pnlPoster0..N` | File | 250KB ea |

**Theming:**
| Field | Type | Notes |
| --------------- | ------ | -------------------------------- |
| `themeCSS` | string | CSS variables to override [default theme](https://raw.githubusercontent.com/OrderlyNetworkDexCreator/dex-creator-template/refs/heads/main/app/styles/theme.css) |
| `tradingViewColorConfig` | string | JSON for chart colors |

**Social:**
| Field | Type | Notes |
| -------------- | ------ | ------------------ |
| `telegramLink` | string | URL |
| `discordLink` | string | URL |
| `xLink` | string | URL |

**Auth/Wallet:**
| Field | Type | Notes |
| ------------------------ | -------- | -------------------------- |
| `walletConnectProjectId` | string | WalletConnect project ID |
| `privyAppId` | string | Privy app ID |
| `privyTermsOfUse` | string | URL to terms |
| `privyLoginMethods` | string | Comma-separated |
| `enableAbstractWallet` | boolean | Enable Abstract wallet |
| `disableEvmWallets` | boolean | Disable EVM wallets |
| `disableSolanaWallets` | boolean | Disable Solana wallets |

**Network:**
| Field | Type | Notes |
| ------------------ | ------- | -------------------- |
| `disableMainnet` | boolean | Disable mainnet |
| `disableTestnet` | boolean | Disable testnet |

**Trading:**
| Field | Type | Notes |
| ------------ | -------------- | ------------------------------- |
| `swapFeeBps` | number (0-100) | Swap fee in basis points (requires "Swap" in enabledMenus) |
| `symbolList` | string | Comma-separated (PERP_ETH_USDC) |

**Menus:**
| Field | Type | Notes |
| -------------- | ------ | ----------------------------- |
| `enabledMenus` | string | Comma-separated. Options: Trading, Portfolio, Markets, Leaderboard (defaults), Swap, Rewards, Vaults, Points |
| `customMenus` | string | Format: "Name,URL;Name2,URL2" |

**SEO:**
| Field | Type | Constraints |
| ------------------- | ------ | --------------------- |
| `seoSiteName` | string | max 100 chars |
| `seoSiteDescription`| string | max 300 chars |
| `seoSiteLanguage` | string | "en" or "en-US" |
| `seoSiteLocale` | string | "en_US" |
| `seoTwitterHandle` | string | "@handle" |
| `seoThemeColor` | string | "#1a1b23" |
| `seoKeywords` | string | max 500 chars |

**Other:**
| Field | Type | Notes |
| -------------------------- | ------- | ------------------------ |
| `availableLanguages` | string | JSON array. Options: en, zh, tc, ja, es, ko, vi, de, fr, ru, id, tr, it, pt, uk, pl, nl |
| `analyticsScript` | string | Base64 encoded |
| `enableServiceDisclaimerDialog` | boolean | Show disclaimer |
| `enableCampaigns` | boolean | Enable ORDER token campaigns and Points menu |
| `restrictedRegions` | string | Comma-separated country names (e.g., "United States,China") |
| `whitelistedIps` | string | IP whitelist |

### Response

**Create (201):** `{ id, brokerId, brokerName, repoUrl, userId, createdAt }`

**Update (200):** Full DEX object with all fields

---

## Key Workflows

### Authentication

Orderly One has its own auth system, separate from the Orderly Network API (`api.orderly.org`). It uses standard EVM `personal_sign` (EIP-191) — **EVM wallets only** (`0x` addresses).

1. `POST /api/auth/nonce` with `{ "address": "0x..." }` → `{ message, nonce }`
2. Sign the returned `message` using `personal_sign` (EIP-191)
3. `POST /api/auth/verify` with `{ "address": "0x...", "signature": "0x..." }` → `{ user: { id, address }, token }`
4. Use `Authorization: Bearer {token}` for all subsequent requests (expires 24h)
5. Validate: `POST /api/auth/validate` with `{ "address", "token" }` → `{ valid: true/false }`

> **Note:** The graduation "finalize admin wallet" step requires authenticating with the Orderly Network API (`api.orderly.org`) — a separate system using EIP-712 or Ed25519 signing. See the graduation workflow below.

### Create DEX Flow

1. Build `multipart/form-data` with fields above
2. `POST /api/dex` → returns `{ id, brokerId, repoUrl }`
3. Poll `GET /api/dex/{id}/workflow-status` until `conclusion: "success"`

### Graduation (Fee Sharing)

**Prerequisites:** DEX created with `brokerId: "demo"` (not already graduated), native USDC on Base, Arbitrum, or Ethereum for the x402 payment.

**Step 1 — Get Fee Options:**

`GET /api/graduation/fee-options` → `{ usdc: { amount }, receiverAddress, payerAddress }`

- Low-code: $100 | Custom SDK: $10 — paid in USDC via x402
- `payerAddress` is the authenticated wallet — it must be the wallet that pays

**Step 2 — Pay via x402 (agent-friendly):**

`POST /api/graduation/graduate?brokerId=…&makerFee=…&takerFee=…&rwaMakerFee=…&rwaTakerFee=…` with `Authorization: Bearer` — pays the fee and registers the broker in a single request.

- First POST returns **402** with a `payment-required` response header (base64 JSON): one EIP-3009 USDC `transferWithAuthorization` option per network (base 8453, arbitrum 42161, ethereum 1)
- Sign the EIP-712 authorization and retry with a `payment-signature` header → **200**: transfer settled on-chain (tx hash in the `PAYMENT-RESPONSE` header, field `.transaction`), broker registered automatically
- Gasless for the payer — the server relays the settlement; invalid params return 400/401 before charging and cancel the payment
- The EIP-3009 payer wallet must be the same wallet as the Bearer-authenticated account
- x402 clients (`@x402/fetch`) default `spendControls.maxAmountPerPayment` to $1 — raise it to cover the fee
- Two-step mode: POST without query params (no Bearer needed — the payment is the auth), then call `verify-tx` (Step 3) with the settlement tx hash and `paymentType: "usdc"`
- If the payment settles but graduation fails, reclaim it: `POST /api/graduation/refund` `{txHash, chain}` (Bearer, payer wallet)

**Step 3 — Verify Transaction (two-step x402 only; skip if auto-finalized):**

`POST /api/graduation/verify-tx` with:

| Field         | Description                                                      |
| ------------- | ---------------------------------------------------------------- |
| `txHash`      | Settlement transaction hash from the `PAYMENT-RESPONSE` header   |
| `chain`       | The chain you paid on: `base`, `arbitrum`, or `ethereum`         |
| `brokerId`    | Desired broker ID (5-15 chars, `^[a-z0-9_-]+$`)                  |
| `makerFee`    | Maker fee in bps (-0.5 to 15, 0.1 increments; negative = rebate) |
| `takerFee`    | Taker fee in bps (1 to 15, 0.1 increments)                       |
| `rwaMakerFee` | RWA maker fee in bps (-0.5 to 15)                                |
| `rwaTakerFee` | RWA taker fee in bps (1 to 15)                                   |
| `paymentType` | `usdc` — pass it explicitly (default is `order`)                 |

The API verifies: tx exists, sender matches authenticated user, recipient is correct, amount meets fee, tx not reused, `brokerId` not taken. The broker ID is created here — but graduation is not finished until Step 4.

**Step 4 — Register Admin Wallet (required to complete graduation):**

**EVM Wallet (agents):**

1. `GET https://api.orderly.org/v1/registration_nonce?address={yourAddress}`
2. Sign EIP-712 typed data: `{ brokerId, chainId, timestamp, registrationNonce }` (same brokerId and address you graduated with)
3. `POST https://api.orderly.org/v1/register_account` with `{ message, signature, userAddress }` → `account_id`
4. `POST /api/graduation/finalize-admin-wallet` with an **empty JSON body `{}`** + Bearer — binds your own wallet as admin (agents omit the multisig fields)

**Solana Wallet:**

1. Same flow but `chainId: 900900900`, sign with Solana wallet, `chainType: "SOL"`

**EVM Multisig/Gnosis Safe:**

1. Safe Wallet → Transaction Builder → batch: `delegateSigner` on Vault contract with `[keccak256(brokerId), userAddress]`
2. Execute with signer approvals
3. `POST /api/graduation/finalize-admin-wallet` with `{ multisigAddress, multisigChainId }`

> **Note:** Step 4's registration authenticates with the Orderly Network API (`api.orderly.org`), not Orderly One — different auth system (EIP-712/Ed25519). `finalize-admin-wallet` itself uses your Orderly One Bearer token. Confirm with `GET /api/graduation/graduation-status` until `isGraduated: true`.

**Graduation Status & Fees:**

| Endpoint                                | Description                                                        |
| --------------------------------------- | ------------------------------------------------------------------ |
| `GET /api/graduation/status`            | Basic status: `{ currentBrokerId, approved }`                      |
| `GET /api/graduation/graduation-status` | Detailed: `{ isGraduated, brokerId, isMultisig, multisigAddress }` |
| `GET /api/graduation/fees`              | Current maker/taker/RWA fee rates                                  |
| `GET /api/graduation/tier`              | Builder Staking Programme tier                                     |
| `POST /api/graduation/refund`           | Reclaim a settled-but-unused x402 payment: `{txHash, chain}`       |

---

## Orderly MCP

This skill references the Orderly MCP server. If not installed, see **orderly-onboarding** skill for setup.

**Tool:** `get_orderly_one_api_info`

- `{ endpoint: "/api/dex" }` - Specific endpoint details
- `{ category: "graduation" }` - All endpoints in a category
- `{}` - Full API overview

---

## Common Issues

| Issue                               | Solution                                                                                                                                     |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| DEX stuck deploying                 | Check `/api/dex/{id}/workflow-runs/{runId}` for job failures                                                                                 |
| Domain not working                  | CNAME to `{org}.github.io`, wait for DNS propagation                                                                                         |
| Graduation verify fails             | Confirm the settlement tx hash and chain from `PAYMENT-RESPONSE`, wait for confirmations                                                     |
| "Broker ID already taken"           | Choose a different `brokerId` in verify-tx                                                                                                   |
| "Transaction hash already used"     | Each tx can only be used once (anti-replay). Send a new payment                                                                              |
| x402 paid but broker not registered | Two-step: `POST /api/graduation/verify-tx` with the settlement tx hash (`paymentType: "usdc"`); if that fails, `POST /api/graduation/refund` |
| x402 payment rejected as too small  | Raise the x402 client `spendControls.maxAmountPerPayment` (default $1) to cover the fee                                                      |
| "Must register EVM address"         | Complete admin wallet registration (Step 4) before calling `finalize-admin-wallet`                                                           |
| "Already graduated"                 | Each user can only graduate once                                                                                                             |
| Logo upload fails                   | Check file size limits (250KB primary, 100KB secondary)                                                                                      |
| Invalid CSS                         | Validate `themeCSS` syntax before submitting                                                                                                 |

---

## Related Skills

- **orderly-onboarding** - Account setup and builder onboarding
- **orderly-trading-orders** - Trading functionality
