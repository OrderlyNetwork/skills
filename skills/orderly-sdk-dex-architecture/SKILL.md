---
name: orderly-sdk-dex-architecture
description: Complete DEX architecture guide including project structure, provider hierarchy, network configuration, TradingView setup, and build/deploy.
---

# Orderly Network: SDK DEX Architecture

A comprehensive guide to architecting and scaffolding a complete DEX application using the Orderly Network Components SDK. All examples below mirror the up-to-date reference template (SDK `3.1.x`).

## When to Use

- Setting up a new DEX project
- Understanding the provider hierarchy
- Configuring network settings and chain filters
- Setting up TradingView charts
- Understanding provider configuration

## Prerequisites

- Node.js 20+ installed (enforced in `package.json` `engines`)
- Orderly SDK packages installed (see `orderly-sdk-install-dependency`)
- React 18+ project with TypeScript
- Vite 7+ build tool

## Overview

This skill covers the complete architecture for building a production-ready DEX:

- Project structure and setup (source dir is `app/`, not `src/`)
- Configuration via Vite `.env` and `import.meta.env`
- Provider hierarchy and configuration
- **Network configuration** — mainnet/testnet, chain filters, default chain
- **TradingView chart setup** — charting library files
- Routing and page components
- Build and deployment (base path, SPA output)

**Critical Configuration**: Every DEX must have:

1. `brokerId` — Your Orderly broker ID (falls back to `"demo"`)
2. `networkId` — Either `"mainnet"` or `"testnet"`
3. Proper wallet connector setup with matching network
4. TradingView charting library in `public/tradingview/` (for chart functionality)

## Project Structure

```
my-dex/
├── public/
│   ├── favicon.webp
│   ├── locales/               # i18n translations
│   │   └── extend/            # Custom translations
│   ├── pnl/                   # PnL share poster backgrounds
│   └── tradingview/           # TradingView library (REQUIRED for charts)
│       ├── chart.css          # Custom chart styles
│       └── charting_library/  # TradingView charting library files
├── app/                       # <-- source root (NOT src/)
│   ├── main.tsx               # Entry point — mounts React (router + providers)
│   ├── App.tsx                # Root layout (provider + <Outlet/>)
│   ├── components/
│   │   ├── orderlyProvider/   # SDK provider setup
│   │   │   ├── index.tsx      # Main provider wrapper
│   │   │   ├── walletConnector.tsx
│   │   │   ├── privyConnector.tsx
│   │   │   └── orderlyLocaleProvider.tsx
│   │   ├── ErrorBoundary.tsx
│   │   └── LoadingSpinner.tsx
│   ├── pages/                 # route components
│   │   ├── perp/              # trading (index + [symbol])
│   │   ├── portfolio/
│   │   ├── markets/
│   │   ├── leaderboard/
│   │   ├── rewards/
│   │   ├── vaults/
│   │   ├── swap/
│   │   └── points/
│   ├── utils/
│   │   ├── config.tsx         # App configuration (nav, scaffold props)
│   │   ├── walletConfig.ts    # Wallet connectors
│   │   ├── theme-config.ts    # resolveDexThemeConfig()
│   │   ├── trading-view-config.ts
│   │   ├── symbol-filter.ts   # createSymbolDataAdapter()
│   │   └── base-path.ts       # withBasePath() helper
│   └── styles/
│       └── index.css          # Global styles + Tailwind
├── .env                       # Environment configuration (VITE_* vars read via import.meta.env)
├── index.html
├── package.json
├── tailwind.config.ts
├── tsconfig.json              # paths alias: "@/*": ["./app/*"]
└── vite.config.ts
```

## Configuration

`OrderlyAppProvider` and the wallet connectors need a handful of values from your app — most importantly:

| Value         | SDK prop       | Notes                                               |
| ------------- | -------------- | --------------------------------------------------- |
| broker ID     | `brokerId`     | Your Orderly broker ID; falls back to `"demo"`      |
| network       | `networkId`    | `"mainnet"` or `"testnet"`                          |
| chain list    | `chainFilter`  | `{ mainnet?: [{id}], testnet?: [{id}] }` (not flat) |
| default chain | `defaultChain` | `{ mainnet: { id } }`                               |
| themes        | `themes`       | Theme array (see **orderly-sdk-theming**)           |
| logos         | `appIcons`     | `AppLogos` (see **orderly-sdk-theming**)            |

**How you source these is up to your app** — a static config module, `.env`, anything. The reference template uses Vite `.env` via `import.meta.env.VITE_*` (inlined at build time):

```bash
# .env (reference template keys — one possible config source)
VITE_ORDERLY_BROKER_ID=your_broker_id        # unset => "demo" broker
VITE_ORDERLY_BROKER_NAME=Your DEX Name
VITE_DEFAULT_CHAIN=42161
VITE_ORDERLY_MAINNET_CHAINS=42161,10,8453
VITE_ORDERLY_TESTNET_CHAINS=421614,84532
VITE_PRIVY_APP_ID=                           # set ONLY to use Privy (see wallet-connection)
VITE_WALLETCONNECT_PROJECT_ID=               # required for WalletConnect / mobile wallets
VITE_RESTRICTED_REGIONS=                     # comma-separated ISO codes (see orderly-one-dex)
VITE_ORDERLY_THEME_CONFIG=                   # JSON theme array (see orderly-sdk-theming)
```

> **No `config.js` / `window.__RUNTIME_CONFIG__`.** A runtime-config indirection only makes sense for multi-instance deployments where one build is reused with different config — an **Orderly One** concern (see `orderly-one-dex`). For a normal DEX, `.env` (or a static config module) is all you need.

### Entry point — `app/main.tsx`

```tsx
// app/main.tsx
import React, { lazy } from 'react';
import ReactDOM from 'react-dom/client';
import { HelmetProvider } from 'react-helmet-async';
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import App from './App';
import './styles/index.css';

// ... lazy page imports ...

const router = createBrowserRouter(
  [
    /* routes */
  ],
  {
    basename: import.meta.env.BASE_URL || '/',
  }
);

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <HelmetProvider>
      <RouterProvider router={router} />
    </HelmetProvider>
  </React.StrictMode>
);
```

## Provider Hierarchy

The SDK requires a specific provider nesting order. `OrderlyLocaleProvider` (i18n) wraps everything; the wallet connector (or Privy) wraps `OrderlyAppProvider`.

```
OrderlyLocaleProvider (i18n)
└── Suspense
    └── WalletConnectorProvider  (or PrivyConnectorProvider)
        └── OrderlyAppProvider
            ├── (internal) AppConfigProvider
            ├── (internal) OrderlyThemeProvider   ← uses the `themes` prop
            ├── (internal) OrderlyConfigProvider (from hooks)
            ├── (internal) AppStateProvider
            ├── (internal) UILocaleProvider
            ├── (internal) TooltipProvider
            ├── (internal) ModalProvider
            └── Your App
```

> `TooltipProvider` and `ModalProvider` are managed internally by `OrderlyAppProvider`. You do **not** add them yourself.

## OrderlyAppProvider Props

The core provider (`@orderly.network/react-app`). Key props:

| Prop             | Type                                     | Notes                                            |
| ---------------- | ---------------------------------------- | ------------------------------------------------ |
| `brokerId`       | `string`                                 | Your broker ID (falls back to `"demo"`)          |
| `brokerName`     | `string`                                 | Display name                                     |
| `networkId`      | `"mainnet" \| "testnet"`                 | Active network                                   |
| `chainFilter`    | `{ mainnet?: [{id}]; testnet?: [{id}] }` | Restrict switchable chains (NOT a flat id array) |
| `defaultChain`   | `{ mainnet: { id } }`                    | Initial chain                                    |
| `themes`         | theme array                              | See **orderly-sdk-theming**                      |
| `appIcons`       | `AppLogos`                               | Logos (see **orderly-sdk-theming**)              |
| `dataAdapter`    | adapter                                  | Custom symbol data adapter                       |
| `onChainChanged` | `(chainId, { isTestnet }) => void`       | Fires when the user switches chain               |
| `restrictedInfo` | `{ customRestrictedRegions?: string[] }` | Geo-restriction (see **orderly-one-dex**)        |

> `TooltipProvider`, `ModalProvider`, `AppConfigProvider`, `OrderlyThemeProvider`, `OrderlyConfigProvider`, and `AppStateProvider` are all mounted **internally** by `OrderlyAppProvider`. Do not add them yourself.

## Minimal Provider

The provider nesting is: i18n (`OrderlyLocaleProvider`) → wallet connector → `OrderlyAppProvider`. Pass neutral config values from your app (sourced however you prefer):

```tsx
// app/components/orderlyProvider/index.tsx
import { ReactNode, useCallback } from 'react';
import { OrderlyAppProvider } from '@orderly.network/react-app';
import type { NetworkId } from '@orderly.network/types';
import { OrderlyLocaleProvider } from './orderlyLocaleProvider';
import { WalletConnector } from './walletConnector';

export function OrderlyProvider({ children }: { children: ReactNode }) {
  const networkId: NetworkId = 'mainnet'; // from your config

  const onChainChanged = useCallback((_chainId: number, { isTestnet }: { isTestnet: boolean }) => {
    // flip networkId + reload on mainnet <-> testnet switches
  }, []);

  return (
    <OrderlyLocaleProvider>
      <WalletConnector networkId={networkId}>
        <OrderlyAppProvider
          brokerId={brokerId} // from your config
          brokerName={brokerName}
          networkId={networkId}
          themes={themes} // see orderly-sdk-theming
          appIcons={appIcons}
          chainFilter={{ mainnet: [{ id: 42161 }, { id: 10 }] }} // optional
          defaultChain={{ mainnet: { id: 42161 } }} // optional
          onChainChanged={onChainChanged}
        >
          {children}
        </OrderlyAppProvider>
      </WalletConnector>
    </OrderlyLocaleProvider>
  );
}
```

> `chainFilter` is `{ mainnet?: [{id}], testnet?: [{id}] }` — **not** a flat array of ids. `defaultChain` is `{ mainnet: { id } }`. Swap the `WalletConnector` for the Privy connector when you want social login (see **orderly-sdk-wallet-connection**).

## Wallet Connector Setup

`WalletConnectorProvider` (`@orderly.network/wallet-connector`) wraps `OrderlyAppProvider`. Both `evmInitial` and `solanaInitial` are **configured** props — they carry the wallet network and the wallet list, so a real DEX passes them rather than omitting them. Disable one side by passing `undefined`:

```tsx
// app/components/orderlyProvider/walletConnector.tsx
import { ReactNode } from 'react';
import { WalletConnectorProvider } from '@orderly.network/wallet-connector';
import { WalletAdapterNetwork } from '@solana/wallet-adapter-base';
import type { NetworkId } from '@orderly.network/types';
import { getEvmInitialConfig, getSolanaWallets } from '@/utils/walletConfig';

const WalletConnector = ({
  children,
  networkId,
  disableEvm,
  disableSolana,
}: {
  children: ReactNode;
  networkId: NetworkId;
  disableEvm?: boolean; // from your config
  disableSolana?: boolean; // from your config
}) => (
  <WalletConnectorProvider
    evmInitial={disableEvm ? undefined : getEvmInitialConfig()}
    solanaInitial={
      disableSolana
        ? undefined
        : {
            network:
              networkId === 'mainnet' ? WalletAdapterNetwork.Mainnet : WalletAdapterNetwork.Devnet,
            wallets: getSolanaWallets(networkId),
          }
    }
  >
    {children}
  </WalletConnectorProvider>
);
export default WalletConnector;
```

> `networkId` must be consistent between `WalletConnectorProvider` (Solana network) and `OrderlyAppProvider`. See **orderly-sdk-wallet-connection** for the `evmInitial` / `solanaInitial` shapes and the Privy connector.

## Network Configuration

### Supported Networks

**Mainnet Chains (Production)**

| Chain    | Chain ID | Description           |
| -------- | -------- | --------------------- |
| Arbitrum | 42161    | Primary mainnet chain |
| Optimism | 10       | OP mainnet            |
| Base     | 8453     | Base mainnet          |
| Ethereum | 1        | Ethereum mainnet      |
| Solana   | N/A      | Solana mainnet        |

**Testnet Chains (Development)**

| Chain            | Chain ID  | Description           |
| ---------------- | --------- | --------------------- |
| Arbitrum Sepolia | 421614    | Primary testnet chain |
| Base Sepolia     | 84532     | Base testnet          |
| Solana Devnet    | 901901901 | Solana devnet         |

### Chain Filter

Restrict which chains users can switch between. `chainFilter` is an object keyed by network, each value an array of `{ id }`:

```tsx
// from VITE_ORDERLY_MAINNET_CHAINS="42161,10,8453"
//   => { mainnet: [{ id: 42161 }, { id: 10 }, { id: 8453 }] }
// from VITE_DEFAULT_CHAIN="42161"
//   => defaultChain = { mainnet: { id: 42161 } }
```

## App Root Component

```tsx
// app/App.tsx
import { Outlet } from 'react-router-dom';
import OrderlyProvider from '@/components/orderlyProvider';

export default function App() {
  return (
    <OrderlyProvider>
      <Outlet />
    </OrderlyProvider>
  );
}
```

`HelmetProvider` and `RouterProvider` live in `main.tsx` (see Configuration). SEO `<Helmet>` and an HTTPS warning are added here in the full template.

## TradingView Chart Setup (REQUIRED for charts)

> **CRITICAL**: The TradingView charting library must be manually added to your `public/tradingview/` folder (it is not distributed via npm).

### Required Files Structure

```
public/
└── tradingview/
    ├── chart.css                    # Optional: custom chart styling
    └── charting_library/            # REQUIRED: TradingView library
        ├── charting_library.js      # Main library script
        ├── charting_library.d.ts
        └── ... (other library files)
```

### How to Get TradingView Library

1. **Request access** from TradingView: https://www.tradingview.com/HTML5-stock-forex-bitcoin-charting-library/
2. **Download** the charting library package
3. **Copy** the `charting_library` folder to your `public/tradingview/` directory

### TradingView Configuration

`TradingPage` accepts a `tradingViewConfig`. In the template it is generated by `createTradingViewConfig(source)` (see **orderly-sdk-theming** for the full color config); inline it looks like:

```tsx
<TradingPage
  symbol={symbol}
  tradingViewConfig={{
    scriptSRC: '/tradingview/charting_library/charting_library.js',
    library_path: '/tradingview/charting_library/',
    customCssUrl: '/tradingview/chart.css',
    colorConfig: { upColor: '#26a69a', downColor: '#ef5350' },
  }}
/>
```

## Tailwind Configuration

```ts
// tailwind.config.ts
import type { Config } from 'tailwindcss';

export default {
  // content globs point at app/, NOT src/. No preset, no custom brand colors.
  content: ['./app/**/{**,.client,.server}/**/*.{js,jsx,ts,tsx}'],
  theme: {
    extend: {
      fontFamily: { sans: ['Inter', 'ui-sans-serif', 'system-ui', 'sans-serif'] },
    },
  },
  plugins: [],
} satisfies Config;
```

```css
/* app/styles/index.css */
@import '@orderly.network/ui/dist/styles.css';
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Vite Configuration

> The wallet connector / Solana packages use Node built-ins (`Buffer`, `crypto`, `stream`). Polyfill **only those three** — do **not** add `util` or `globals` (they break the build). Some deps ship CommonJS that must be interop'd via `vite-plugin-cjs-interop`. Path aliases come from `tsconfig.json` via `vite-tsconfig-paths` (no manual `resolve.alias`).

```bash
npm install -D vite-plugin-node-polyfills vite-plugin-cjs-interop vite-tsconfig-paths
```

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';
import { cjsInterop } from 'vite-plugin-cjs-interop';
import { nodePolyfills } from 'vite-plugin-node-polyfills';

export default defineConfig(() => ({
  base: process.env.PUBLIC_PATH || '/', // base path for sub-path deploys
  plugins: [
    react(),
    tsconfigPaths(), // reads "@/*" from tsconfig.json
    cjsInterop({ dependencies: ['bs58', '@coral-xyz/anchor', 'lodash'] }),
    nodePolyfills({ include: ['buffer', 'crypto', 'stream'] }), // NO util, NO globals
  ],
  build: { outDir: 'build/client' },
  optimizeDeps: { include: ['react', 'react-dom', 'react-router-dom'] },
}));
```

`tsconfig.json` path alias:

```json
{ "compilerOptions": { "paths": { "@/*": ["./app/*"] } } }
```

## Checklist for Production

- [ ] Broker ID configured in `.env` (or intentionally using `"demo"`)
- [ ] WalletConnect project ID in `.env` (for mobile wallets)
- [ ] TradingView library in `public/tradingview/` (if using charts)
- [ ] Chain filter / default chain set in `.env`
- [ ] Custom branding (logo, favicon, theme — see orderly-sdk-theming)
- [ ] Error tracking (Sentry, etc.)
- [ ] Analytics integration
- [ ] SSL/HTTPS enabled
- [ ] `PUBLIC_PATH` set for sub-path deploys

## Related Skills

- **orderly-sdk-install-dependency** - SDK package installation
- **orderly-sdk-wallet-connection** - Wallet integration details
- **orderly-sdk-page-components** - Using pre-built page components
- **orderly-sdk-trading-workflows** - End-to-end trading implementation
- **orderly-sdk-theming** - Customizing the UI appearance
- **orderly-sdk-debugging** - Troubleshooting common issues
