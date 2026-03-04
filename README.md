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

| Category            | Skill                            | Description                                                                                                         |
| ------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Getting Started** | `orderly-onboarding`             | **START HERE** - Agent onboarding, MCP server setup, skills overview, and developer quickstart                      |
| **API / Protocol**  | `orderly-api-authentication`     | Complete authentication - EIP-712 (EVM) or Ed25519 message (Solana) signing for account/keys, Ed25519 for API calls |
|                     | `orderly-trading-orders`         | Place, manage, and cancel orders via REST API or SDK hooks                                                          |
|                     | `orderly-positions-tpsl`         | Monitor positions, configure TP/SL, manage leverage                                                                 |
|                     | `orderly-websocket-streaming`    | Real-time WebSocket connections for orderbook, executions, positions                                                |
|                     | `orderly-deposit-withdraw`       | Token deposits, withdrawals, and cross-chain operations                                                             |
| **SDK / React**     | `orderly-sdk-react-hooks`        | Reference for all Orderly React SDK hooks                                                                           |
|                     | `orderly-ui-components`          | Pre-built React UI components for trading interfaces                                                                |
|                     | `orderly-sdk-install-dependency` | Install Orderly SDK packages and related dependencies                                                               |
|                     | `orderly-sdk-dex-architecture`   | Complete DEX project structure, provider hierarchy, and configuration                                               |
|                     | `orderly-sdk-page-components`    | Pre-built page components (TradingPage, Portfolio, Markets)                                                         |
|                     | `orderly-sdk-theming`            | CSS variable theming and customization                                                                              |
|                     | `orderly-sdk-trading-workflows`  | End-to-end trading flows (connect → deposit → trade → withdraw)                                                     |
| **Platform**        | `orderly-sdk-wallet-connection`  | Wallet integration for EVM and Solana chains                                                                        |
|                     | `orderly-sdk-debugging`          | Debug and troubleshoot SDK errors                                                                                   |
|                     | `orderly-one-dex`                | Create and manage custom DEX with Orderly One API                                                                   |

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

- **Building a DEX**: orderly-sdk-install-dependency, orderly-sdk-dex-architecture, orderly-sdk-wallet-connection, orderly-sdk-trading-workflows
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

### orderly-sdk-install-dependency

SDK package installation:

- Core packages (react-app, hooks, types, ui)
- Feature widgets (trading, portfolio, markets)
- Wallet connectors (EVM and Solana)
- UI sub-packages for custom integrations
- Tailwind CSS and Vite polyfill setup

### orderly-sdk-dex-architecture

DEX architecture and setup:

- Project structure and file organization
- Provider hierarchy and configuration
- Network configuration (mainnet/testnet)
- TradingView chart setup
- Runtime configuration

### orderly-sdk-page-components

Pre-built page components:

- TradingPage - Full trading interface
- Portfolio pages (Overview, Positions, Orders, Assets, History)
- MarketsHomePage - Markets listing
- LeaderboardPage - Trading competitions
- Router setup and customization

### orderly-sdk-theming

UI customization:

- CSS variable system for colors and styling
- Brand colors, semantic colors, trading colors
- Custom fonts and typography
- TradingView chart colors
- PnL share backgrounds

### orderly-sdk-trading-workflows

End-to-end trading flows:

- Connect wallet and authenticate
- Deposit funds
- Place orders (market, limit)
- Monitor positions and orders
- Close positions and withdraw

### orderly-sdk-wallet-connection

Wallet integration:

- EVM wallet support (MetaMask, WalletConnect, etc.)
- Solana wallet support (Phantom, Solflare)
- Chain switching and network management
- Account state management
- Privy integration for social login

### orderly-sdk-debugging

Debugging and troubleshooting:

- Build and setup errors
- Common API error codes
- WebSocket connection monitoring
- Account state issues
- Order submission debugging

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
│   ├── orderly-onboarding/
│   │   └── SKILL.md
│   ├── orderly-api-authentication/
│   │   └── SKILL.md
│   └── ... (14 more skills)
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
