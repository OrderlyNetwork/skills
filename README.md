# Orderly Network Skills

AI Agent Skills for Orderly Network - Perpetual Futures Trading Infrastructure.

## Installation

Install all skills with the `skills` CLI:

```bash
npx skills add orderly-network/orderly-skills
```

Install specific skills:

```bash
# List available skills first
npx skills add orderly-network/orderly-skills --list

# Install specific skill
npx skills add orderly-network/orderly-skills --skill orderly-trading-orders

# Install multiple skills
npx skills add orderly-network/orderly-skills --skill orderly-getting-started --skill orderly-api-authentication

# Install all skills
npx skills add orderly-network/orderly-skills --skill '*'
```

## Available Skills

| Skill                         | Description                                                                                |
| ----------------------------- | ------------------------------------------------------------------------------------------ |
| `orderly-getting-started`     | Complete onboarding workflow - wallet connection, account registration, API key generation |
| `orderly-trading-orders`      | Place, manage, and cancel orders via REST API or SDK hooks                                 |
| `orderly-positions-tpsl`      | Monitor positions, configure TP/SL, manage leverage                                        |
| `orderly-websocket-streaming` | Real-time WebSocket connections for orderbook, executions, positions                       |
| `orderly-deposit-withdraw`    | Token deposits, withdrawals, and cross-chain operations                                    |
| `orderly-sdk-react-hooks`     | Reference for all Orderly React SDK hooks                                                  |
| `orderly-ui-components`       | Pre-built React UI components for trading interfaces                                       |
| `orderly-api-authentication`  | Ed25519 signature authentication for REST API                                              |
| `orderly-one-dex`             | Create and manage custom DEX with Orderly One API                                          |

## Skill Details

### orderly-getting-started

Complete onboarding workflow covering:

- Wallet connection (EVM & Solana)
- EIP-712 signing for account registration
- Ed25519 key pair generation
- API key registration
- Quick setup with React SDK

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

### orderly-api-authentication

Authentication implementation:

- Ed25519 key generation
- Message construction
- Signature creation
- Header formatting
- Complete API client example

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
│   ├── orderly-getting-started/
│   │   └── SKILL.md
│   ├── orderly-trading-orders/
│   │   └── SKILL.md
│   └── ... (7 more skills)
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
