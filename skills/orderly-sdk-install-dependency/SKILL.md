---
name: orderly-sdk-install-dependency
description: Install Orderly SDK packages and related dependencies (hooks, UI, features, wallet connectors) using the preferred package manager.
---

# Orderly Network: SDK Install Dependency

Use this skill to add Orderly SDK packages to your project. The SDK is modular—install only what you need. Versions below reflect the current `3.1.x` line.

## When to Use

- Starting a new DEX project
- Adding Orderly SDK to an existing project
- Installing specific packages for custom integrations
- Setting up wallet connectors

## Prerequisites

- Node.js 20+ installed (enforced in `package.json` `engines`)
- npm, yarn, or pnpm package manager
- React 18+ project (for UI packages)

## Quick Start (Full DEX)

> **IMPORTANT**: A functional DEX requires BOTH the Orderly packages AND the wallet connector dependencies. `@orderly.network/wallet-connector` needs `@web3-onboard/*` packages for EVM wallets and `@solana/wallet-adapter-*` packages for Solana wallets. `<WalletConnectorProvider>` takes **configured** `evmInitial` / `solanaInitial` props — they are not optional in a real DEX (see **orderly-sdk-wallet-connection**).
>
> **Note**: `@orderly.network/hooks` is a transitive dependency of `@orderly.network/react-app` — you do not need to install it separately unless you are on the hooks-only integration path.

```bash
# Orderly SDK packages (all pinned to the same 3.1.x release)
npm install @orderly.network/react-app \
            @orderly.network/trading \
            @orderly.network/portfolio \
            @orderly.network/markets \
            @orderly.network/wallet-connector \
            @orderly.network/i18n

# REQUIRED: EVM wallet support (MetaMask, WalletConnect, Binance, etc.)
npm install @web3-onboard/injected-wallets @web3-onboard/walletconnect \
            @binance/w3w-blocknative-connector wagmi

# REQUIRED: Solana wallet support (Phantom, Solflare, mobile, etc.)
npm install @solana/wallet-adapter-base @solana/wallet-adapter-wallets \
            @solana-mobile/wallet-adapter-mobile
```

## Package Reference

### Core Packages

| Package                      | Description                                           | Key Exports                                                                                                                    |
| ---------------------------- | ----------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `@orderly.network/react-app` | Main app provider, config context                     | `OrderlyAppProvider`, `useAppContext`, `useAppConfig`                                                                          |
| `@orderly.network/hooks`     | React hooks for trading, account, orders, positions   | `useAccount`, `useOrderEntry`, `usePositionStream`, `useOrderStream`, `useDeposit`, `useWithdraw`, `useLeverage`, `useMarkets` |
| `@orderly.network/types`     | TypeScript type definitions and constants             | `API`, `OrderType`, `OrderSide`, `OrderStatus`, `NetworkId`, `ChainConfig`                                                     |
| `@orderly.network/ui`        | Base UI components (buttons, inputs, dialogs, tables) | `Button`, `Input`, `Dialog`, `Table`, `Tabs`, `Select`, `Tooltip`, `Modal`, `Spinner`, `toast`, `useScreen`, `Flex`, `cn`      |
| `@orderly.network/i18n`      | Internationalization (i18n) support                   | `LocaleProvider`, `useTranslation`, `i18n`, `defaultLanguages`                                                                 |
| `@orderly.network/utils`     | Utility functions (formatting, math, dates)           | `formatNumber`, `Decimal`, `dayjs`                                                                                             |

```bash
npm install @orderly.network/react-app @orderly.network/hooks @orderly.network/types @orderly.network/ui @orderly.network/i18n
```

## Feature Widgets (High-Level Pages)

Complete, pre-built page components with full functionality. **Exports below are the real names imported by the reference template** — note `Dashboard.DashboardPage`, `GeneralLeaderboardWidget`, and `PointSystemPage` specifically.

| Package                                | Description                                                  | Key Exports (verified)                                                                                                    |
| -------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| `@orderly.network/trading`             | Full trading page (chart, orderbook, order entry, positions) | `TradingPage`, `OrderBook`, `LastTrades`, `AssetView`, `RiskRate`, `SplitLayout`                                          |
| `@orderly.network/portfolio`           | Portfolio dashboard (positions, orders, assets, history)     | `OverviewModule`, `PositionsModule`, `OrdersModule`, `AssetsModule`, `HistoryModule`, `FeeTierModule`, `APIManagerModule` |
| `@orderly.network/markets`             | Markets listing page with prices and stats                   | `MarketsHomePage`, `MarketsProvider`, `MarketsList`, `SymbolInfoBar`, `FundingOverview`                                   |
| `@orderly.network/vaults`              | Vault/Earn products page                                     | `VaultsPage`                                                                                                              |
| `@orderly.network/affiliate`           | Referral/affiliate program                                   | `Dashboard` (use `Dashboard.DashboardPage`), `ReferralProvider`                                                           |
| `@orderly.network/trading-leaderboard` | Trading competition leaderboard                              | `GeneralLeaderboardWidget`                                                                                                |
| `@orderly.network/trading-points`      | Points/merits program page                                   | `PointSystemPage`                                                                                                         |

```bash
npm install @orderly.network/trading @orderly.network/portfolio @orderly.network/markets @orderly.network/trading-leaderboard @orderly.network/trading-points @orderly.network/affiliate @orderly.network/vaults
```

> There is no `@orderly.network/trading-rewards` package. The points/rewards page is `@orderly.network/trading-points` exporting `PointSystemPage`. Likewise `@orderly.network/affiliate` does **not** export `AffiliatePage` — use `Dashboard.DashboardPage` inside `<ReferralProvider>`.

## Wallet Connectors

Choose **one** wallet connection strategy.

| Package                                   | Description                                         | Key Exports                    |
| ----------------------------------------- | --------------------------------------------------- | ------------------------------ |
| `@orderly.network/wallet-connector`       | Standard connector (Web3-Onboard + Solana adapters) | `WalletConnectorProvider`      |
| `@orderly.network/wallet-connector-privy` | Privy connector (social login, embedded wallets)    | `WalletConnectorPrivyProvider` |

**Option A: Standard Wallet Connector (Recommended)**

Supports EVM (MetaMask, WalletConnect, Binance, etc.) and Solana (Phantom, Solflare, Ledger, mobile).

> The connector requires the underlying wallet packages — you build `evmInitial` (injected + WalletConnect connectors) and `solanaInitial` (network + wallet list) and pass both as props. See **orderly-sdk-wallet-connection** for the full `walletConfig.ts`.

```bash
npm install @orderly.network/wallet-connector

# EVM wallets
npm install @web3-onboard/injected-wallets @web3-onboard/walletconnect \
            @binance/w3w-blocknative-connector wagmi

# Solana wallets
npm install @solana/wallet-adapter-base @solana/wallet-adapter-wallets \
            @solana-mobile/wallet-adapter-mobile
```

**Option B: Privy Connector**

For social login (Google, X, email, passkey) and embedded wallets.

> The Privy package depends on `@privy-io/cross-app-connect` (NOT the older `@privy-io/react-auth`).

```bash
npm install @orderly.network/wallet-connector-privy @privy-io/cross-app-connect
```

## UI Sub-Packages (Granular Components)

Individual UI modules for custom integrations. These are dependencies of `@orderly.network/trading` and `@orderly.network/portfolio`, but can be installed separately. In the reference template, these are consumed via `@orderly.network/ui` and `@orderly.network/ui-scaffold` re-exports rather than imported directly.

| Package                              | Description                                   | Key Exports                                                                                                |
| ------------------------------------ | --------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `@orderly.network/ui-scaffold`       | App layout scaffold, navigation, account menu | `Scaffold`, `MainNavWidget`, `BottomNavWidget`, `AccountMenuWidget`, `ChainMenuWidget`, `SideNavbarWidget` |
| `@orderly.network/ui-order-entry`    | Order entry form component                    | `OrderEntry`                                                                                               |
| `@orderly.network/ui-positions`      | Positions table component                     | `PositionsView`                                                                                            |
| `@orderly.network/ui-orders`         | Orders table component                        | `OrdersView`                                                                                               |
| `@orderly.network/ui-transfer`       | Deposit/withdraw/transfer dialogs             | `DepositWidget`, `WithdrawWidget`                                                                          |
| `@orderly.network/ui-leverage`       | Leverage selector component                   | `LeverageWidget`                                                                                           |
| `@orderly.network/ui-tpsl`           | Take profit / stop loss form                  | `TPSLWidget`                                                                                               |
| `@orderly.network/ui-share`          | PnL sharing card generator                    | `SharePnL`                                                                                                 |
| `@orderly.network/ui-chain-selector` | Chain/network selector dropdown               | `ChainSelector`                                                                                            |
| `@orderly.network/ui-connector`      | Wallet connect button & modal                 | `ConnectWalletButton`                                                                                      |
| `@orderly.network/ui-tradingview`    | TradingView chart wrapper                     | `TradingViewChart`                                                                                         |
| `@orderly.network/ui-notification`   | Notification center                           | `NotificationWidget`                                                                                       |

```bash
npm install @orderly.network/ui-scaffold @orderly.network/ui
```

## Low-Level Packages

For advanced customization or non-React environments.

| Package                                   | Description                                          | Key Exports                                                   |
| ----------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------- |
| `@orderly.network/core`                   | Low-level API client, signing, key management        | `Account`, `ConfigStore`, `WalletAdapter`, `signMessage`      |
| `@orderly.network/perp`                   | Perpetual trading calculations (margin, liquidation) | `calcMargin`, `calcLiqPrice`, `calcPnL`, `calcIMR`, `calcMMR` |
| `@orderly.network/net`                    | Network/WebSocket layer                              | `WebSocketClient`, `EventEmitter`                             |
| `@orderly.network/default-evm-adapter`    | Default EVM wallet adapter                           | `EVMAdapter`                                                  |
| `@orderly.network/default-solana-adapter` | Default Solana wallet adapter                        | `SolanaAdapter`                                               |

```bash
npm install @orderly.network/core @orderly.network/perp
```

## Installation by Use Case

### Minimal Setup (Hooks Only)

For building fully custom UI with hooks.

```bash
npm install @orderly.network/react-app \
            @orderly.network/hooks \
            @orderly.network/types \
            @orderly.network/wallet-connector
```

### Full DEX with All Features

Mirrors the reference template's dependencies:

```bash
npm install @orderly.network/react-app \
            @orderly.network/trading \
            @orderly.network/portfolio \
            @orderly.network/markets \
            @orderly.network/vaults \
            @orderly.network/affiliate \
            @orderly.network/trading-leaderboard \
            @orderly.network/trading-points \
            @orderly.network/wallet-connector \
            @orderly.network/wallet-connector-privy \
            @orderly.network/i18n \
            @orderly.network/types \
            @orderly.network/ui \
            @orderly.network/ui-scaffold
```

The reference template also pulls in `woofi-swap-widget-kit` (for the swap page), `wagmi`, `react-helmet-async` (SEO), `react-router-dom` (v7, routing), `firebase`, `zod`, `ip-range-check`, and `@plausible-analytics/tracker`. Add these only if your build uses those features.

### Privy (Social Login) Setup

```bash
npm install @orderly.network/react-app \
            @orderly.network/trading \
            @orderly.network/wallet-connector-privy \
            @privy-io/cross-app-connect
```

## Peer Dependencies

All packages require:

```json
{
  "peerDependencies": {
    "react": ">=18",
    "react-dom": ">=18"
  }
}
```

## Tailwind CSS Setup

The UI packages require Tailwind CSS. There is **no Orderly preset** — point the content glob at `app/` (the source root) and the `@orderly.network` packages.

```bash
npm install -D tailwindcss postcss autoprefixer
```

**tailwind.config.ts:**

```ts
import type { Config } from 'tailwindcss';

export default {
  content: ['./app/**/{**,.client,.server}/**/*.{js,jsx,ts,tsx}'],
  theme: { extend: {} },
  plugins: [],
} satisfies Config;
```

**CSS entry file:**

> Only `@orderly.network/ui` ships a CSS file. `trading`, `portfolio`, `markets`, etc. reuse these base styles — do not look for per-package CSS.

```css
/* app/styles/index.css */
@import '@orderly.network/ui/dist/styles.css';

@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Vite Plugins (Required)

The wallet connector / Solana packages use Node.js built-ins (`Buffer`, `crypto`, `stream`) and some ship CommonJS. Polyfill **only `buffer`, `crypto`, `stream`** — do **not** add `util` or `globals` (they break the build). Interop CommonJS deps with `vite-plugin-cjs-interop`. Path aliases come from `tsconfig.json` via `vite-tsconfig-paths`.

```bash
npm install -D vite-plugin-node-polyfills vite-plugin-cjs-interop vite-tsconfig-paths
```

**vite.config.ts:**

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';
import { cjsInterop } from 'vite-plugin-cjs-interop';
import { nodePolyfills } from 'vite-plugin-node-polyfills';

export default defineConfig({
  plugins: [
    react(),
    tsconfigPaths(),
    cjsInterop({ dependencies: ['bs58', '@coral-xyz/anchor', 'lodash'] }),
    nodePolyfills({ include: ['buffer', 'crypto', 'stream'] }), // NO util, NO globals
  ],
});
```

## Version Compatibility

All `@orderly.network/*` packages must share the **same version**. The exception is `@orderly.network/trading-points`, which follows its own version line. Current reference versions:

```json
{
  "dependencies": {
    "@orderly.network/react-app": "3.1.6",
    "@orderly.network/trading": "3.1.6",
    "@orderly.network/portfolio": "3.1.6",
    "@orderly.network/hooks": "3.1.6",
    "@orderly.network/ui": "3.1.6",
    "@orderly.network/trading-points": "2.0.2"
  }
}
```

## Package Manager Commands

**npm:**

```bash
npm install <package-name>
```

**yarn:**

```bash
yarn add <package-name>
```

**pnpm:**

```bash
pnpm add <package-name>
```

## Related Skills

- **orderly-sdk-dex-architecture** - Project structure and provider setup
- **orderly-sdk-wallet-connection** - Wallet integration details
- **orderly-sdk-page-components** - Using pre-built page components
- **orderly-sdk-trading-workflows** - End-to-end trading implementation
- **orderly-sdk-theming** - Customizing the UI appearance
