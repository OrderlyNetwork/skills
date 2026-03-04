# Orderly Network Skills

AI Agent Skills for Orderly Network - Perpetual Futures Trading Infrastructure.

## Installation

Install all skills with the `skills` CLI:

```bash
npx skills add @orderly.network/skills
```

Install specific skills:

```bash
# List available skills first
npx skills add @orderly.network/skills --list

# Install specific skill
npx skills add @orderly.network/skills --skill orderly-trading-orders

# Install multiple skills
npx skills add @orderly.network/skills --skill orderly-api-authentication --skill orderly-trading-orders

# Install all skills
npx skills add @orderly.network/skills --skill '*'
```

## Available Skills

| Skill                         | Description                                                                                                         |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `orderly-onboarding`          | **START HERE** - Agent onboarding, MCP server setup, skills overview, and developer quickstart                      |
| `orderly-api-authentication`  | Complete authentication - EIP-712 (EVM) or Ed25519 message (Solana) signing for account/keys, Ed25519 for API calls |
| `orderly-trading-orders`      | Place, manage, and cancel orders via REST API or SDK hooks                                                          |
| `orderly-positions-tpsl`      | Monitor positions, configure TP/SL, manage leverage                                                                 |
| `orderly-websocket-streaming` | Real-time WebSocket connections for orderbook, executions, positions                                                |
| `orderly-deposit-withdraw`    | Token deposits, withdrawals, and cross-chain operations                                                             |
| `orderly-sdk-react-hooks`     | Reference for all Orderly React SDK hooks                                                                           |
| `orderly-ui-components`       | Pre-built React UI components for trading interfaces                                                                |
| `orderly-one-dex`             | Create and manage custom DEX with Orderly One API                                                                   |

## Skill Details

### orderly-onboarding

**START HERE** if you're new to Orderly Network.

This skill provides:

- **Overview**: What Orderly is and how it works
- **Architecture**: How the omnichain orderbook infrastructure fits together
- **AI Agent Tools**: MCP server installation and agent skills setup
- **Getting Started**: Directives for builders, API developers, and DEX creators
- **Key Links**: Documentation, SDK, example DEX, Discord

**Install MCP Server:**

```bash
npx @orderly.network/mcp-server init --client <claude|cursor|vscode|codex|opencode>
```

**Load Recommended Skills:**

- **Building a DEX**: orderly-sdk-install-dependency, orderly-sdk-dex-architecture, orderly-sdk-wallet-connection
- **API/Bot Development**: orderly-api-authentication, orderly-trading-orders
- **Custom DEX (Orderly One)**: orderly-one-dex

### orderly-api-authentication

Complete two-layer authentication system:

**Wallet Authentication (Layer 1):**

- EVM: EIP-712 wallet signatures for account registration and key management
- Solana: Ed25519 message signing for account registration and key management

**API Authentication (Layer 2):**

- Ed25519 key pair generation
- Ed25519 request signing for REST API (EVM and Solana)
- WebSocket authentication

**Environment & Configuration:**

- Environment configuration (mainnet/testnet)
- Supported chains reference (EVM and Solana)
- Contract addresses (vault, verifying contracts)
- Ed25519 request signing for REST API
- WebSocket authentication
- Environment configuration (mainnet/testnet)
- Supported chains reference

### orderly-trading-orders

Order management including:

- Order types (LIMIT, MARKET, IOC, FOK, POST_ONLY)
- REST API endpoints
- React SDK useOrderEntry hook
- Batch operations
- Order validation

### orderly-positions-tpsl

Position and risk management:

- Position streaming
- Unrealized PnL tracking
- Take-Profit / Stop-Loss orders
- Leverage settings
- Position closing

### orderly-websocket-streaming

Real-time data streaming:

- Public streams (orderbook, trades, klines)
- Private streams (execution reports, positions, balance)
- Authentication flow
- Connection management

### orderly-deposit-withdraw

Asset management:

- Token deposits via vault
- Withdrawal flow with signing
- Cross-chain transfers
- Internal transfers

### orderly-sdk-react-hooks

Complete hook reference:

- Account hooks (useAccount, useWalletConnector)
- Trading hooks (useOrderEntry, useOrderStream)
- Position hooks (usePositionStream, useTPSLOrder)
- Market data hooks (useOrderbookStream, useMarkPrice)
- Balance hooks (useCollateral, useBalance)

### orderly-ui-components

Pre-built components:

- OrderEntry
- Positions table
- Orderbook
- WalletConnect
- Charts (TradingView, Lightweight)
- Tables, Sheets, Modals

### orderly-one-dex

Custom DEX management:

- DEX creation and deployment
- Custom domain setup
- Theme customization
- Graduation (demo to full DEX)
- Leaderboard integration

## Development

### Prerequisites

- Node.js 20+
- Yarn

### Setup

```bash
yarn install
```

### Validate Skills

```bash
yarn validate
```

### Lint

```bash
yarn lint
yarn lint:fix
```

### Format

```bash
yarn format
yarn format:check
```

## Project Structure

```
orderly-skills/
├── skills/
│   ├── orderly-api-authentication/
│   │   └── SKILL.md
│   ├── orderly-trading-orders/
│   │   └── SKILL.md
│   └── ... (6 more skills)
├── scripts/
│   └── validate-skills.ts
├── package.json
├── tsconfig.json
├── eslint.config.mjs
├── .prettierrc
└── README.md
```

## Links

- [Orderly Documentation](https://orderly.network/docs)
- [Orderly SDK](https://github.com/OrderlyNetwork/js-sdk)
- [Orderly Discord](https://discord.gg/OrderlyNetwork)
- [Skills Documentation](https://skills.sh)

## License

MIT
