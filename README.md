# Orderly Network Skills

AI Agent Skills for Orderly Network - Perpetual Futures Trading Infrastructure.

## Installation

Install Orderly skills to enhance your AI agent with procedural knowledge for building on Orderly.

### Quick Install (All Skills)

**Global** (available across all projects):

```bash
npx skills add OrderlyNetwork/skills --all -g
```

**Local** (project-specific):

```bash
npx skills add OrderlyNetwork/skills --all
```

### Target Specific Agents

Install for all agents:

```bash
npx skills add OrderlyNetwork/skills --all --agent '*' -g
```

Install for specific agents:

```bash
# Claude Code
npx skills add OrderlyNetwork/skills --all --agent claude-code -g

# Cursor
npx skills add OrderlyNetwork/skills --all --agent cursor -g

# OpenCode
npx skills add OrderlyNetwork/skills --all --agent opencode -g

# VS Code
npx skills add OrderlyNetwork/skills --all --agent vscode -g

# Codex
npx skills add OrderlyNetwork/skills --all --agent codex -g
```

### Install Specific Skills

List available skills:

```bash
npx skills add OrderlyNetwork/skills --list
```

Install specific skills:

```bash
npx skills add OrderlyNetwork/skills --skill orderly-trading-orders --skill orderly-api-authentication
```

Install all skills in a category:

```bash
npx skills add OrderlyNetwork/skills --skill '*'
```

### Global vs Local Installation

**Global (`-g` flag):**

- Skills available across all your projects
- Installed to user-level directory
- Good for agents working across multiple repos
- Use when: You work on multiple Orderly projects

**Local (no flag):**

- Skills specific to current project
- Creates `.skills/` directory in repo
- Can be committed to version control
- Use when: Team consistency, project-specific setup

### Recommended Setup for AI Agents

**Complete Setup:**

```bash
# 1. Install MCP server for documentation access
npx @orderly.network/mcp-server init --client <your-client>

# 2. Install all skills globally for all agents
npx skills add OrderlyNetwork/skills --all --agent '*' -g
```

**Minimal Setup:**

```bash
# Install only onboarding skill, let agent guide you to others
npx skills add OrderlyNetwork/skills --skill orderly-onboarding -g
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
|                     | `orderly-sdk-plugins`            | SDK v3 plugins, interceptors, and Module Marketplace workflows                                                      |
| **Platform**        | `orderly-sdk-wallet-connection`  | Wallet integration for EVM and Solana chains                                                                        |
|                     | `orderly-sdk-debugging`          | Debug and troubleshoot SDK errors                                                                                   |
| **Orderly One**     | `orderly-one-general`            | Overview of Orderly One white-label DEX platform - launch paths, authentication, API base URLs                      |
|                     | `orderly-one-create-dex`         | Create, update, delete, and deploy a DEX through the Orderly One REST API                                           |
|                     | `orderly-one-dex`                | Create and manage custom DEX with Orderly One API                                                                   |
|                     | `orderly-one-graduation`         | Graduate a demo DEX into a fee-earning DEX with broker ID creation and payment                                      |
|                     | `orderly-one-theming`            | Generate and customize DEX themes using AI or manual CSS via the Orderly One API                                    |
|                     | `orderly-one-template`           | Customize the forked DEX template repository (React + Vite + Orderly SDK)                                           |
|                     | `orderly-one-leaderboard`        | Read-only endpoints for DEX rankings and platform statistics                                                        |

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
- **Custom DEX (Orderly One)**: orderly-one-general, orderly-one-create-dex, orderly-one-graduation, orderly-one-theming, orderly-one-template

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
- Isolated margin and `order_tag` custom fee integration

### orderly-positions-tpsl

Position and risk management:

- Position streaming
- Unrealized PnL tracking
- Take-Profit / Stop-Loss orders
- Leverage settings
- Position closing
- Isolated margin position adjustments

### orderly-websocket-streaming

Real-time data streaming:

- Public streams (orderbook, trades, klines)
- Private streams (execution reports, positions, balance)
- Authentication flow
- Connection management

### orderly-deposit-withdraw

Asset management:

- Token deposits via vault
- Exclusive receiver address deposits
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

### orderly-one-general

Orderly One platform overview:

- What Orderly One is and how it works
- Two launch paths (low-code vs. custom SDK/API)
- Authentication and API base URLs
- Fee structure and graduation overview

### orderly-one-create-dex

DEX creation via REST API:

- Create, update, delete DEX via API
- Multipart/form-data requests with file uploads
- Integration type, branding, and blockchain configuration
- Deployment management

### orderly-one-graduation

Graduation process:

- Convert demo DEX to fee-earning DEX
- Broker ID creation
- Payment in USDC, USDT, or ORDER tokens
- Admin wallet registration

### orderly-one-theming

Theme customization:

- AI-driven theme generation from text descriptions
- Manual CSS editing via API
- Multiple theme variants
- Live preview via web portal

### orderly-one-template

Template repository customization:

- Fork and customize the DEX template repo
- React + Vite + Orderly SDK stack
- GitHub Pages deployment via GitHub Actions
- Direct frontend code modifications

### orderly-one-leaderboard

DEX rankings and statistics:

- Public read-only endpoints (no auth required)
- Sorting by volume, PnL, or fees
- Time period filtering
- Platform-wide statistics

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
│   └── ... (20 more skills)
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
