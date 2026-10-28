---
name: orderly-onboarding
description: Agent onboarding for Orderly Network - infrastructure overview, agent tools (CLI, MCP, SDK), quickstart, broker ID creation, supported chains, and key links.
---

# Orderly Network: Agent Onboarding

Orderly is an omnichain orderbook-based trading infrastructure providing perpetual futures liquidity for decentralized exchanges. This skill is your starting point for building on or learning about Orderly Network.

## When to Use

- First time encountering Orderly Network
- Creating a broker ID to earn trading-fee revenue
- Setting up AI agent tools for Orderly development
- Understanding the Orderly ecosystem and offerings
- Finding the right skill or resource for your task
- Trading or managing an account from the terminal

## What is Orderly Network

Orderly is a combination of an orderbook-based trading infrastructure and a robust liquidity layer offering perpetual futures orderbooks. Unlike traditional platforms, Orderly doesn't have a front end—it operates at the core of the ecosystem, providing essential services to projects built on top.

**Key Characteristics:**

- **Omnichain CLOB**: Shared Central Limit Order Book accessible from all major EVM chains and Solana
- **Backend Infrastructure**: No official front end; builders create DEXes and trading interfaces on top
- **On-chain Settlement**: All trades settle on-chain while maintaining full self-custody
- **Unified Liquidity**: One orderbook serves all integrated front-ends
- **Perpetual Futures**: Trade BTC, ETH, SOL, and more with up to 50x leverage
- **Gasless Trading**: No gas fees once funds are deposited and trading keys activated
- **One-Click Trading**: New trading key pair per session, no further signatures needed

**Primary Use Cases:**

| Use Case              | Description                                                                |
| --------------------- | -------------------------------------------------------------------------- |
| **Builders/DEXes**    | Create your own Perps DEX on EVM and Solana with plug-and-play SDKs        |
| **Perps Aggregators** | Access Orderly's shared liquidity directly via API or SDK                  |
| **Trading Desks**     | Use APIs for CEX-level trading with low latency orderbook                  |
| **Trading Bots**      | Connect to orderbook for best rates, SL/limit orders, gasless transactions |
| **AI Agents**         | Operate accounts and trades through the `orderly` CLI                      |

## Key Advantages

- **Unified Orderbook & Liquidity**: Access all major chains through a single trading infrastructure
- **Quick Development**: Launch a DEX within days using our SDKs
- **Ready-to-Use Liquidity**: Powered by multiple top-tier market makers
- **Revenue Sharing**: Earn a share of generated fees from your platform
- **CEX-Level Performance**: Low latency matching engine with on-chain settlement
- **Self-Custody**: You control your assets and private keys
- **Collaborative Ecosystem**: Join a thriving community of builders

## Architecture

Your Application (DEX, Bot, Wallet, Aggregator)

- Orderly Infrastructure
  - **CLOB** — Shared Central Limit Order Book (unified across all chains)
  - **Matching Engine** — Low-latency order matching (CEX-level performance)
  - **Vault** — On-chain settlement with self-custody
  - **Risk Management** — Liquidation engine and position monitoring
- Settlement Networks
  - **EVM**: Arbitrum, Optimism, Base, Ethereum, Polygon, Mantle
  - **Non-EVM**: Solana

## Getting Started: AI Agent Tools

There are three agent surfaces. Use each for what it is best at:

| Surface            | Use it for                                                            |
| ------------------ | --------------------------------------------------------------------- |
| **Orderly CLI**    | Live operations: accounts, orders, positions, funds, market data      |
| **Orderly MCP**    | Reference: docs, REST/WS endpoints, SDK symbols, contracts, workflows |
| **Orderly skills** | Procedural knowledge and guardrails (this skill and **orderly-sdk**)  |

### Orderly MCP Server

The MCP server is the reference layer. Connect to the **remote server** at `https://mcp.orderly.network` (Streamable HTTP, no auth, no installation), or **install it locally** via npx:

```bash
npx @orderly.network/mcp-server init --client <client>
```

If automatic setup doesn't work, manually configure your AI client to run `npx @orderly.network/mcp-server@latest`.

**What the MCP Server Provides (both remote and local):**

| Tool                          | Description                                        |
| ----------------------------- | -------------------------------------------------- |
| `search_orderly_docs`         | Search Orderly docs and type-accurate SDK symbols  |
| `get_contract_addresses`      | Lookup smart contract addresses for any chain      |
| `explain_workflow`            | Step-by-step guides for common tasks               |
| `get_api_info`                | REST API and WebSocket endpoint documentation      |
| `get_indexer_api_info`        | Trading metrics, events, volume statistics         |
| `get_component_guide`         | React UI component building guides                 |
| `get_orderly_one_api_info`    | DEX creation and management API for Orderly One    |
| `get_strategy_vault_api_info` | Strategy Vault API: yield strategies and providers |
| `get_public_info_api_info`    | Zero-auth Public Info API: market/account data     |

MCP resources are available too: `orderly://overview`, `orderly://workflows`, `orderly://sdk/hooks`, `orderly://sdk/components`, `orderly://api/rest`, `orderly://api/websocket`, `orderly://contracts`.

### Agent Skills

Install Orderly skills to enhance your AI agent with procedural knowledge for building on Orderly.

**Install all skills globally (recommended):**

```bash
npx skills add OrderlyNetwork/skills --all --agent '*' -g
```

**Install all skills locally:**

```bash
npx skills add OrderlyNetwork/skills --all
```

**Install specific skills:**

```bash
# List available skills
npx skills add OrderlyNetwork/skills --list

# Install both skills
npx skills add OrderlyNetwork/skills --skill orderly-onboarding --skill orderly-sdk

# Install for a specific agent
npx skills add OrderlyNetwork/skills --all --agent claude-code -g
```

**Available Skills:**

| Category            | Skill                | Description                                                                                             |
| ------------------- | -------------------- | ------------------------------------------------------------------------------------------------------- |
| **Getting Started** | `orderly-onboarding` | This skill — overview, agent tools, CLI operations, broker IDs, ecosystem reference                     |
| **SDK / React**     | `orderly-sdk`        | Build a DEX: template-first setup, packages/providers, wallet connection, UI, hooks, theming, debugging |

## For Builders (SDK & DEX Development)

Build trading interfaces using Orderly's React SDK (3.x).

> **Default rule for agents:** when a user wants a DEX, start from the **[DEX Template](https://github.com/OrderlyNetwork/dex-template)** — fork, configure, deploy. Only build from scratch with the SDK packages if the user explicitly needs something the template can't provide (custom layouts, non-trading features, hooks-level control). If the requirement is ambiguous, ask the user before writing code.

### Fastest Way to Get Started (Default)

Fork the **[DEX Template](https://github.com/OrderlyNetwork/dex-template)** to get a working DEX in minutes.

1. Fork the repository to your GitHub account
2. Configure `.env` with your broker ID and name
3. Customize your theme (CSS variables and color tokens)
4. Deploy to your preferred hosting platform

This template uses the **components SDK** — pre-built page components that work out of the box with less customization. Building from scratch is opt-in: load **orderly-sdk** for the package/import map, provider hierarchy, wallet setup, hooks, and theming, and use the MCP server for exact signatures.

**Core SDK Packages:**

```bash
# Full DEX setup
npm install @orderly.network/react-app \
            @orderly.network/trading \
            @orderly.network/portfolio \
            @orderly.network/markets \
            @orderly.network/wallet-connector \
            @orderly.network/i18n

# Required: EVM wallet support
npm install @web3-onboard/injected-wallets @web3-onboard/walletconnect

# Required: Solana wallet support
npm install @solana/wallet-adapter-base @solana/wallet-adapter-wallets
```

**Key Components Available:**

- `TradingPage` - Full trading page (chart, orderbook, order entry, positions)
- `OrderEntryWidget` - Order placement form
- `OrderBook` - Market depth display
- `PositionsWidget` - Position management table
- `Scaffold` - App layout, navigation, account menu
- `ConnectWalletButton` - Wallet connection UI

**Orderly One (White-Label DEX):**

Launch your own branded perpetuals DEX without building from scratch. Orderly One provides a turnkey solution with:

- Custom domain and branding
- Fee revenue sharing after paying the graduation fee
- Full trading infrastructure
- Custom theme

**Load for SDK development:**

- **orderly-sdk** - the complete build guide
- MCP `search_orderly_docs` / `get_component_guide` for exact symbols and patterns

## For API / Bot Developers

Integrate directly with Orderly's REST API and WebSocket streams.

**API Base URLs:**

| Network | URL                               |
| ------- | --------------------------------- |
| Mainnet | `https://api.orderly.org`         |
| Testnet | `https://testnet-api.orderly.org` |

**WebSocket URLs:**

| Network | URL                               |
| ------- | --------------------------------- |
| Mainnet | `wss://ws.orderly.org/ws`         |
| Testnet | `wss://testnet-ws.orderly.org/ws` |

**Authentication:**

- Ed25519 key pair generation for API signing
- EIP-712 wallet signatures for EVM accounts
- Ed25519 message signing for Solana accounts

Use MCP `get_api_info` with type `auth` for the signing recipe, or let the **orderly CLI** handle keys and signing entirely.

**Public Data Without Auth:**

The Public Info API is a single zero-auth endpoint (`POST /v1/public/query`) covering 24 query types across `market`, `account`, `platform`, and `system` categories (e.g., `marketSummary`, `accountState`, `orderbook`, `candles`, `topAddresses`, `whaleContext`). Use it for bots and reads that don't need authenticated trading — pass a `type` field and params, no signing required. Query it via the `get_public_info_api_info` MCP tool for exact params, response fields, weights, and examples per query type.

**Symbol Format:**

```
PERP_<TOKEN>_USDC
```

Examples: `PERP_ETH_USDC`, `PERP_BTC_USDC`, `PERP_SOL_USDC`

**Key Endpoints:**

- `POST /v1/order` - Place order
- `GET /v1/positions` - Get positions
- `GET /v1/orders` - Get orders
- `GET /v1/orderbook/{symbol}` - Orderbook snapshot
- `GET /v1/public/futures` - Market info

For endpoint detail, WebSocket streams, and signing examples, query MCP `get_api_info` rather than memorizing them.

## Orderly CLI

`orderly` is the agent-safe terminal client for Orderly Network, distributed as a single static binary — no Node.js runtime required. Wallet and API keys live in the OS keychain, signing happens inside the binary, and agents only ever see results. JSON is the default output; `--csv` is available for list data. Newer builds expose `orderly schema --json` (a machine-readable manifest that marks mutating commands) and per-command `--dry-run`.

**Install** — one line for Linux and macOS (detects the architecture and verifies the release checksum):

```bash
curl -fsSL https://orderly.network/cli.sh | sh
orderly --version
```

The binary is installed to `~/.local/bin`; override with `ORDERLY_INSTALL_DIR`, or pin a release with `ORDERLY_VERSION`. Require **2.x** — `1.x` is the retired Node CLI. Windows: download the `orderly-*-pc-windows-msvc.zip` asset from the [latest release](https://github.com/OrderlyNetwork/cli/releases/latest) and add it to `PATH`. See [INSTALL.md](https://github.com/OrderlyNetwork/cli/blob/master/INSTALL.md) for checksums and signatures.

**Quick Start (Testnet):**

```bash
orderly wallet-create --type EVM
orderly wallet-register --broker-id demo
orderly faucet-usdc <address> --broker-id demo --chain-id 421614
orderly wallet-add-key --broker-id demo --scope read,trading
orderly auth-list
orderly account-balance --account <account-id>
orderly order-place PERP_ETH_USDC BUY MARKET 0.01 --account <account-id>
```

Faucet delivery is delayed; check the balance later. `wallet-add-key` remembers the network you used as the default.

**Key rules:**

- **`--account` is required** for authenticated commands. Use `orderly auth-list` (`--ids` for scripting).
- **Quote hex account IDs** — `--account "0x5a6b…"`; unquoted hex is parsed as a number.
- **Symbols are uppercase** — `PERP_ETH_USDC`, not `PERP_ETH_usdc`.
- **Default network is testnet** — pass `--network mainnet` for production.
- **Output** — compact JSON by default, `--csv` for lists, `--pretty` for readable JSON.
- **Linux keychain** needs a running Secret Service (gnome-keyring/KWallet).
- **`auth-export-key` blocks agents** — it requires an interactive TTY and typing `EXPORT`. Never attempt it.

**Command map:**

| Area          | Commands                                                                                                                                           |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| Setup / auth  | `wallet-create`, `wallet-import`, `wallet-list`, `wallet-register`, `wallet-add-key`, `auth-import`, `auth-list`, `auth-show`                      |
| Account       | `account-info`, `account-balance`, `account-statistics`, `account-limits`, `fee-tier`, `key-info`, `account-liquidations`                          |
| Orders        | `order-place`, `order-list`, `order-get`, `order-edit`, `order-cancel`, `order-cancel-all`, `order-cancel-by-client-id`, `order-trades`            |
| Batch orders  | `batch-order-place`, `batch-order-cancel`, `batch-order-cancel-by-client-id`                                                                       |
| Algo / TP-SL  | `algo-order-place`, `algo-order-list`, `algo-order-edit`, `algo-order-cancel`, `algo-order-cancel-all`, `algo-order-get`, `algo-order-trades`      |
| Positions     | `positions-list`, `positions-close`, `positions-history`, `leverage`, `trades`, `funding-history`                                                  |
| Market data   | `market-price`, `market-orderbook`, `market-trades`, `symbols`, `kline`, `funding-rates`, `price-changes`, `open-interest`, `liquidated-positions` |
| Funds         | `chains`, `tokens`, `deposit-info`, `withdraw`, `asset-history`, `faucet-usdc` (testnet)                                                           |
| PnL / rewards | `settle-pnl`, `settle-pnl-history`, `referral-info`, `distribution-history`, `volume-stats`, `notification-inbox`, `notification-unread`           |
| Orderly One   | `broker-create`, `broker-status`                                                                                                                   |

Run `orderly --help`, `orderly <command> --help`, or `orderly schema --json` (newer builds) for the authoritative surface. Algo types: `STOP`, `TP_SL`, `POSITIONAL_TP_SL`, `TRAILING_STOP`, `BRACKET`.

**Safety:** read-only commands are safe. Confirm with the user before placing/cancelling orders, closing positions, changing leverage, withdrawing, settling PnL, or removing/revoking keys. Prefer `--dry-run` where available. Never move real funds on mainnet without explicit user approval.

**Common errors:**

| Symptom                                   | Fix                                                                  |
| ----------------------------------------- | -------------------------------------------------------------------- |
| `401`                                     | API key not registered or wrong network — run `wallet-add-key`       |
| `403`                                     | Signature invalid/expired — check the system clock                   |
| Empty account list                        | Wrong network, or no key stored — `auth-list`, then `wallet-add-key` |
| `Account not found` / missing `--account` | Run `auth-list` and pass the ID (quoted if hex)                      |
| Keychain error on Linux                   | Start gnome-keyring/KWallet (Secret Service)                         |

## Broker IDs and Fee Revenue

Your broker ID is how Orderly attributes trading volume to you — once graduated, you earn a share of the fees generated by trades routed through your broker ID. This is the revenue-generating step for builders.

| Option            | Cost               | Setup                                          | Revenue     |
| ----------------- | ------------------ | ---------------------------------------------- | ----------- |
| **`demo`**        | Free               | None — use `demo` in your config               | None        |
| **Custom broker** | 10 USDC (one-time) | CLI `broker-create`, or Orderly One API + x402 | Fee sharing |

**CLI path (interactive):**

```bash
orderly broker-create my-broker --address 0xYourEvmWallet --network mainnet
orderly broker-status --address 0xYourEvmWallet --network mainnet
```

`broker-create` authenticates the EVM wallet, creates a custom-integration DEX record if needed, shows the fee and receiver, then verifies the payment tx and registers the wallet as broker admin. The operator sends the exact amount and pastes the tx hash, so this needs a TTY.

### Create a Custom Broker ID via x402

**Prerequisites:** an EVM wallet holding native USDC on Base (8453), Arbitrum (42161), or Ethereum (1). The payment is gasless for you — you sign an EIP-3009 authorization off-chain and the server relays the settlement, so no ETH is needed. Base URL: `https://dex-api.orderly.network` (testnet: `https://testnet-dex-api.orderly.network` for a cheap dry run). The API's OpenAPI spec (`https://dex-api.orderly.network/openapi.json`) embeds an agent quickstart with this same flow.

| #   | Step                                             | Call                                                                                                                                    |
| --- | ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Authenticate (EIP-191 `personal_sign`, EVM only) | `POST /api/auth/nonce` `{address}` → sign the returned message → `POST /api/auth/verify` `{address, signature}` → Bearer JWT (24h)      |
| 2   | Create DEX record                                | `POST /api/dex` (multipart): `brokerName` (3-30 chars), `integrationType: "custom"` — no hosted frontend needed                         |
| 3   | Check the fee                                    | `GET /api/graduation/fee-options` → `{ usdc: { amount }, receiverAddress, payerAddress }` — `payerAddress` must be your wallet          |
| 4   | Pay fee + register broker (one request)          | `POST /api/graduation/graduate?brokerId=…&makerFee=…&takerFee=…&rwaMakerFee=…&rwaTakerFee=…` with Bearer — x402 payment, auto-finalizes |
| 5   | Register wallet as broker admin                  | `GET https://api.orderly.org/v1/registration_nonce` → EIP-712 sign → `POST /v1/register_account`                                        |
| 6   | Finalize                                         | `POST /api/graduation/finalize-admin-wallet` with empty body `{}` + Bearer                                                              |
| 7   | Verify                                           | `GET /api/graduation/graduation-status` until `isGraduated: true`; `GET /api/leaderboard/broker/{brokerId}`                             |

**The x402 step (4) in detail:** the first POST returns **402** with a `payment-required` response header (base64-encoded JSON) containing one EIP-3009 USDC `transferWithAuthorization` option per accepted network (base, arbitrum, ethereum). Sign the EIP-712 authorization with your wallet and retry the same request with a `payment-signature` header → **200**: the transfer settles on-chain (tx hash in the `PAYMENT-RESPONSE` response header) and the broker is registered automatically. Invalid parameters are rejected **before** charging — 400/401 and the payment is canceled, nothing is paid. With an x402 client such as `@x402/fetch` this is a single `fetch` call, but its default `spendControls.maxAmountPerPayment` is $1 — raise it to cover the fee.

**Parameters:** `brokerId` 5-15 chars, `^[a-z0-9_-]+$`. Fees in bps with 0.1 increments: `makerFee`/`rwaMakerFee` -0.5 to 15 (negative = rebates), `takerFee`/`rwaTakerFee` 1 to 15. Example: maker 0 / taker 5.

**Two-step mode & recovery:** POST `graduate` without query params (no Bearer needed — the payment itself is the authentication), then `POST /api/graduation/verify-tx` with the settlement tx hash and `paymentType: "usdc"`. If a payment settles but graduation fails, reclaim it via `POST /api/graduation/refund` `{txHash, chain}` (Bearer, payer wallet).

Humans can also do all of this through the web wizard at [dex.orderly.network](https://dex.orderly.network). For the full graduation reference (Solana/multisig admin wallets, fee tiers, refund recovery), query MCP `get_orderly_one_api_info` with category `graduation`. Agents may create broker IDs but must not deploy or manage DEX frontends.

## Supported Chains

Orderly supports multiple EVM and non-EVM chains. To get the current list of supported networks with their chain IDs, vault addresses, and RPC endpoints:

```
GET https://api.orderly.org/v1/public/chain_info
```

This endpoint returns all mainnet and testnet chains currently supported by Orderly, including Arbitrum, Optimism, Base, Ethereum, Polygon, Mantle, Solana, Sei, Avalanche, BSC, Abstract, and more. The CLI exposes the same data as `orderly chains` and `orderly tokens`.

## $ORDER Token

The $ORDER token is central to the Orderly ecosystem:

- **Maximum Supply:** 1,000,000,000 tokens
- **Staking:** Stake $ORDER to earn VALOR and protocol revenue share
- **VALOR:** Non-transferable metric measuring staking position; redeemable for esORDER rewards
- **Revenue Sharing:** 30% of protocol net fees distributed to stakers
- **Governance:** Stakers participate in protocol governance decisions
- **esORDER:** Escrowed ORDER for rewards with vesting mechanics

**Token Contracts:**

| Network          | Address                                        |
| ---------------- | ---------------------------------------------- |
| Ethereum (ERC20) | `0xABD4C63d2616A5201454168269031355f4764337`   |
| EVM Chains (OFT) | `0x4E200fE2f3eFb977d5fd9c430A41531FB04d97B8`   |
| Solana           | `ABt79MkRXUsoHuV2CVQT32YMXQhTparKFjmidQxgiQ6E` |

For full tokenomics details, visit: https://orderly.network/docs/introduction/tokenomics

## Key Links

| Resource              | URL                                                       | Notes                              |
| --------------------- | --------------------------------------------------------- | ---------------------------------- |
| Documentation         | https://orderly.network/docs                              |                                    |
| SDK Repository        | https://github.com/orderlynetwork/js-sdk                  |                                    |
| DEX Template          | https://github.com/OrderlyNetwork/dex-template            | Fastest start, uses components SDK |
| MCP Server (Remote)   | https://mcp.orderly.network                               | No install needed                  |
| MCP Server (npm)      | https://www.npmjs.com/package/@orderly.network/mcp-server | Local install                      |
| CLI (binary releases) | https://github.com/OrderlyNetwork/cli/releases/latest     |                                    |
| CLI install script    | https://orderly.network/cli.sh                            | Linux and macOS                    |
| Skills.sh             | https://skills.sh                                         |                                    |
| DEX Dashboard         | https://dex.orderly.network                               |                                    |
| Orderly One API spec  | https://dex-api.orderly.network/openapi.json              | Agent quickstart inside            |
| Orderly App           | https://app.orderly.network                               |                                    |
| Discord               | https://discord.gg/OrderlyNetwork                         |                                    |
| Twitter               | https://twitter.com/OrderlyNetwork                        |                                    |

## Recommended Next Steps

**If you want to earn trading-fee revenue:**

1. Create a broker ID via `orderly broker-create` or the x402 flow above
2. Set your `brokerId` in your app, bot, or SDK config so trades attribute fees to you
3. If the user also wants a DEX frontend, default to the [DEX Template](https://github.com/OrderlyNetwork/dex-template)

**If you're building a DEX:**

1. Default: fork the **[DEX Template](https://github.com/OrderlyNetwork/dex-template)** — a working DEX in minutes, no SDK wiring needed
2. Install the MCP server: `npx @orderly.network/mcp-server init` or connect to `https://mcp.orderly.network`
3. Configure your broker settings in `.env` and customize your theme
4. Build from scratch only if the template can't do it — load **orderly-sdk** and ask the user first
5. Set up wallet connection with **orderly-sdk**

**If you're building trading bots or API integrations:**

1. Use the **orderly CLI** for execution (it owns keys and signing)
2. Use MCP `get_api_info` for REST/WebSocket details if you need a custom client
3. Use MCP `get_public_info_api_info` for zero-auth reads
4. Use the CLI for testnet funding (`orderly faucet-usdc`) while developing

**If you want to trade from the terminal:**

1. Install the CLI: `curl -fsSL https://orderly.network/cli.sh | sh`
2. Run `orderly --help` and follow the testnet quick start above
3. Use `--broker-id demo` for testing, or create your own broker ID

**If you're launching a white-label DEX:**

1. Connect to the MCP server for Orderly One tools: `https://mcp.orderly.network`
2. Query `get_orderly_one_api_info` for DEX creation, graduation, and theme endpoints
3. Load **orderly-sdk** for theme structure and frontend work

**If you're troubleshooting:**

1. Search the MCP docs with `search_orderly_docs`
2. For SDK build/runtime issues, load **orderly-sdk**
3. For CLI errors, see the common errors table above

**For testing:**

- Use testnet for development
- Request test USDC with `orderly faucet-usdc` (testnet only)
- Each account can use the faucet up to 3 times

## Common Issues

### "Where do I start building?"

**Default — quick DEX**: Fork the [DEX Template](https://github.com/OrderlyNetwork/dex-template), configure `.env`, and deploy. Uses pre-built components — fastest path, and the right default for most users.

**Only if the user needs more control**: Set up the MCP server — either connect to `https://mcp.orderly.network` (remote, no install) or install locally with `npx @orderly.network/mcp-server init --client <your-client>`. Then load **orderly-sdk** to build with the hooks SDK. If it's unclear which the user wants, ask before building from scratch.

### "What's the difference between MCP server and Skills?"

- **MCP Server**: Runtime tools for your AI assistant (documentation search, pattern lookup, API reference) — available as a remote server at `https://mcp.orderly.network` or installable locally via npx
- **Skills**: Procedural knowledge embedded in your context (how-to guides, guardrails, defaults)

Use both for the best experience.

### "How do I test without real funds?"

Use the testnet environment:

- CLI: `orderly faucet-usdc <address> --broker-id demo --chain-id 421614`
- API: `https://testnet-api.orderly.org`
- WebSocket: `wss://testnet-ws.orderly.org/ws`
- Faucet endpoint: `POST https://testnet-operator-evm.orderly.org/v1/faucet/usdc`

### "Do I need to handle authentication manually?"

No. The SDK and the CLI handle authentication automatically. Only a custom REST client needs the raw signing flow — query MCP `get_api_info` with type `auth`, or use `orderly auth-import` / `wallet-add-key` and let the CLI sign.

### "How do I get a custom broker ID?"

Two paths: `orderly broker-create` (interactive, manual payment tx) or the x402 flow (fully programmatic, gasless EIP-3009 payment). Both are described in **Broker IDs and Fee Revenue** above. A testnet dry run is available.

## Related Skills

- **orderly-sdk** — building a DEX or trading UI with the React SDK.

For everything else — raw API signing, endpoint specs, WebSocket streams, SDK hook signatures, contract addresses, deposit/withdraw internals, Orderly One endpoints — use the Orderly MCP rather than a memory dump in a skill.
