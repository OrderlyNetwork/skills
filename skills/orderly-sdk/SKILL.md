---
name: orderly-sdk
description: Build an Orderly perpetuals DEX with the React SDK - template-first setup, packages and providers, wallet connection, UI components, hooks, theming, and debugging.
---

# Orderly Network: SDK

Use this skill to build a DEX or trading interface on Orderly. For live trading from an agent — orders, balances, positions, funds — use the `orderly` CLI instead (see **orderly-onboarding**).

> This skill carries the decisions and pitfalls that docs lookups get wrong. For complete APIs, signatures, and workflows, query the Orderly MCP; do not duplicate them here.

## Default: fork the template

Fork **[dex-template](https://github.com/OrderlyNetwork/dex-template)** and configure it. It is a working components-SDK DEX, not a starter shell.

1. Fork/clone the repo.
2. Set `.env`: `VITE_ORDERLY_BROKER_ID`, `VITE_ORDERLY_BROKER_NAME`, `VITE_DEFAULT_CHAIN`, `VITE_ORDERLY_MAINNET_CHAINS`, `VITE_ORDERLY_TESTNET_CHAINS`, `VITE_WALLETCONNECT_PROJECT_ID`.
3. Customize the theme (see **Theming**).
4. Deploy, then verify the broker with `orderly broker-status --address 0x…`.

Build from scratch only when the template cannot meet the requirement (custom layouts, non-trading features, hooks-level control). If the requirement is ambiguous, ask the user before writing SDK wiring.

## Versions and packages

- Current line: SDK **3.x**. All `@orderly.network/*` packages must share the same version; the exception is `@orderly.network/trading-points`, which follows its own line.
- Do not hardcode patch versions in code or docs. Resolve the current version with `npm view @orderly.network/react-app version` or MCP `search_orderly_docs`.
- `@orderly.network/hooks` is a transitive dependency of `react-app`; install it explicitly only for a hooks-only app.

```bash
# Components-SDK DEX
npm install @orderly.network/react-app @orderly.network/trading \
            @orderly.network/portfolio @orderly.network/markets \
            @orderly.network/wallet-connector @orderly.network/i18n

# Wallet connector internals (required in practice)
npm install @web3-onboard/injected-wallets @web3-onboard/walletconnect \
            @binance/w3w-blocknative-connector wagmi
npm install @solana/wallet-adapter-base @solana/wallet-adapter-wallets \
            @solana-mobile/wallet-adapter-mobile
```

Privy (social login) replaces the standard connector:

```bash
npm install @orderly.network/wallet-connector-privy @privy-io/cross-app-connect
```

## Where components live

Imports matter. There is **no `@orderly.network/react` package** — that import is a common generated-code error.

| What you need              | Package                                                    | Export                                                |
| -------------------------- | ---------------------------------------------------------- | ----------------------------------------------------- |
| App provider               | `@orderly.network/react-app`                               | `OrderlyAppProvider`, `ErrorBoundary`                 |
| Full trading page          | `@orderly.network/trading`                                 | `TradingPage`, `OrderBook`, `LastTrades`              |
| Order entry                | `@orderly.network/ui-order-entry`                          | `OrderEntryWidget` (self-contained)                   |
| Positions table            | `@orderly.network/ui-positions`                            | `PositionsWidget`                                     |
| Orders table               | `@orderly.network/ui-orders`                               | `OrdersView`                                          |
| Deposit / withdraw dialogs | `@orderly.network/ui-transfer`                             | `DepositWidget`, `WithdrawWidget`                     |
| Leverage / TP-SL           | `@orderly.network/ui-leverage`, `@orderly.network/ui-tpsl` | `LeverageWidget`, `TPSLWidget`                        |
| App layout                 | `@orderly.network/ui-scaffold`                             | `Scaffold`, `MainNavWidget`, `AccountMenuWidget`      |
| Wallet button / auth gate  | `@orderly.network/ui-connector`                            | `ConnectWalletButton`, `AuthGuard`                    |
| Chart                      | `@orderly.network/ui`                                      | `TradingView`                                         |
| Base primitives            | `@orderly.network/ui`                                      | `Button`, `Input`, `Table`, `Modal`, `Sheet`, `toast` |
| Portfolio pages            | `@orderly.network/portfolio`                               | `OverviewModule`, `PositionsModule`, …                |
| Markets page               | `@orderly.network/markets`                                 | `MarketsHomePage`, `MarketsProvider`                  |
| Hooks                      | `@orderly.network/hooks`                                   | `useAccount`, `useOrderEntry`, `useDeposit`, …        |

`@orderly.network/ui-orderbook` and `@orderly.network/ui-chart` do **not** exist. Orderbook lives in `@orderly.network/trading`; the chart is `TradingView` in `@orderly.network/ui`.

## Provider hierarchy

`OrderlyLocaleProvider` → wallet connector (or Privy) → `OrderlyAppProvider` → your app. The wallet connector must wrap `OrderlyAppProvider`, not the other way around.

```tsx
import { OrderlyAppProvider } from '@orderly.network/react-app';
import { WalletConnectorProvider } from '@orderly.network/wallet-connector';

function App() {
  return (
    <WalletConnectorProvider evmInitial={evmInitial} solanaInitial={solanaInitial}>
      <OrderlyAppProvider
        brokerId="your_broker_id"
        brokerName="Your DEX"
        networkId="mainnet"
        themes={themes}
        appIcons={appIcons}
      >
        <YourApp />
      </OrderlyAppProvider>
    </WalletConnectorProvider>
  );
}
```

- Do **not** add `QueryClientProvider`, `ModalProvider`, `TooltipProvider`, `AppConfigProvider`, `OrderlyThemeProvider`, `OrderlyConfigProvider`, or `AppStateProvider` — `OrderlyAppProvider` mounts them internally.
- There is no `TradingPageProvider` or `SymbolProvider`; `TradingPage` manages symbol context itself.
- `chainFilter` is `{ mainnet?: [{ id }], testnet?: [{ id }] }`, not a flat ID array. `defaultChain` is `{ mainnet: { id } }`.

## Wallet connection

The standard connector takes configured `evmInitial` and `solanaInitial` props; they are not optional in a real DEX. Build them in a `walletConfig.ts` module (the template has one) and keep the Solana network consistent with `OrderlyAppProvider.networkId`.

`useWalletConnector()` returns a **flat** object — `connect`, `disconnect`, `connecting`, `wallet`, `connectedChain`, `setChain`, `namespace` — not nested under `wallet`.

For Privy social login, swap `WalletConnectorProvider` for `WalletConnectorPrivyProvider` and set `VITE_PRIVY_APP_ID`. Full provider props and per-wallet config: MCP `search_orderly_docs` with query `wallet connector`.

## Pages and components

Prefer page-level components over hand-assembling widgets:

```tsx
import { TradingPage } from '@orderly.network/trading';

<TradingPage symbol="PERP_ETH_USDC" tradingViewConfig={tradingViewConfig} />;
```

- `Scaffold` (`@orderly.network/ui-scaffold`) provides nav, account menu, and layout.
- `Portfolio` modules: `OverviewModule`, `PositionsModule`, `OrdersModule`, `AssetsModule`, `HistoryModule`, `FeeTierModule`, `APIManagerModule`.
- Markets: `MarketsHomePage`. Vaults: `VaultsPage` from `@orderly.network/vaults`.
- Leaderboard: `GeneralLeaderboardWidget` from `@orderly.network/trading-leaderboard`. Points: `PointSystemPage` from `@orderly.network/trading-points`. Affiliate: `Dashboard.DashboardPage` inside `ReferralProvider` from `@orderly.network/affiliate`.

For the full page list, routing pattern, and props, use MCP `search_orderly_docs` or copy the template's `app/pages/` structure.

## Hooks

The SDK exposes hooks for hooks-level control. Do not reimplement what a hook already provides.

| Area           | Hooks                                                                     |
| -------------- | ------------------------------------------------------------------------- |
| Account / auth | `useAccount`, `useWalletConnector`                                        |
| Orders         | `useOrderEntry`, `useOrderStream`                                         |
| Positions      | `usePositionStream`, `useTPSLOrder`                                       |
| Market data    | `useOrderbookStream`, `useMarkPrice`, `useTickerStream`, `useSymbolsInfo` |
| Balances       | `useCollateral`, `useHoldingStream`                                       |
| Funds          | `useDeposit`, `useWithdraw`                                               |
| Risk           | `useLeverage`, `useMarginRatio`, `useChains`                              |

Critical return shapes:

- `useAccount()` → `{ account, state }`; `state.status` is a numeric `AccountStatusEnum` from `@orderly.network/types` (`SignedIn` is required for private data and orders).
- `useOrderEntry(symbol)` → `formattedOrder`, `metaState`, `helper`, `maxQty`, `estLiqPrice`, `submit`. `metaState.errors === null` means the order validates; there is no `isSubmitting`.
- `useOrderStream({ status: OrderStatus.INCOMPLETE })` — use the enum, not the string `'OPEN'`.

Full signatures and examples: MCP resource `orderly://sdk/hooks` or `search_orderly_docs` with `scope: "sdk"`.

## Theming

Pass 1–6 theme objects to `OrderlyAppProvider`'s `themes` prop. Exactly one must set `isDefault`.

```ts
const themes = [
  {
    id: '11111111-1111-4111-8111-111111111111',
    displayName: 'Dark',
    mode: 'dark',
    isDefault: true,
    cssVars: {
      '--oui-color-primary': '#29dfa9',
      '--oui-color-base-1': '#0e0e10',
      '--oui-color-trading-profit': '#29dfa9',
      '--oui-color-trading-loss': '#f5618b',
    },
    tradingViewColorConfig: { chartBG: '#0e0e10', upColor: '#29dfa9', downColor: '#f5618b' },
  },
];
```

- Variables use the `--oui-*` prefix (SDK v2.8+; v1 used `--orderly`). Any subset is valid.
- Categories: brand, semantic, surfaces/text (`--oui-color-base-1…10`), PnL, gradients, radius/spacing/font.
- Chart colors belong in `tradingViewColorConfig` on the theme; library paths belong in `tradingViewConfig` on `TradingPage`.
- Key logos via `appIcons` (`AppLogos`), not by editing package CSS.

Full variable list and PnL share posters: MCP `explain_workflow("theming")`.

## Build configuration

- **Tailwind:** no Orderly preset. Point `content` at `app/**/*.{ts,tsx}` and import `@orderly.network/ui/dist/styles.css` in the CSS entry. Other feature packages reuse those base styles; do not look for per-package CSS.
- **Vite:** polyfill only `buffer`, `crypto`, and `stream`. Adding `util` or `globals` breaks the build. Interop CommonJS deps with `vite-plugin-cjs-interop`, and load path aliases with `vite-tsconfig-paths`.
- **TradingView:** charts need the TradingView library files served by your app. Copy the template's chart setup; the API does not ship them.
- **React:** 18+ for the components SDK.

## Debugging

| Symptom                         | Check                                                                                                    |
| ------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Blank app                       | `OrderlyAppProvider` mounted with valid `brokerId`/`networkId`; `usePreLoadData().done`; `useWsStatus()` |
| Private data empty              | `state.status === SignedIn`; correct network; WebSocket connected                                        |
| Sign-in loops                   | Wallet connector mounted and configured; user rejected the signature                                     |
| Orders rejected                 | `metaState.errors === null`; symbol exact; quantity ≥ min; price in range; free collateral               |
| Deposit fails                   | ERC-20 allowance first; correct vault address for the chain                                              |
| Withdraw fails                  | Amount ≤ `maxAmount`; unsettled PnL accounted; wallet prompt accepted                                    |
| `createClient` / `.mjs` errors  | Bundler interop and Vite polyfills (see **Build configuration**)                                         |
| Wrong imports / missing exports | Check the package table above; there is no `@orderly.network/react`                                      |

For a stuck account, log `AccountStatusEnum[state.status]`; for HTTP failures, inspect the SDK's `ApiError` (`status`, `code`, `message`).

## Look things up in the MCP

| Need                                | Call                                                          |
| ----------------------------------- | ------------------------------------------------------------- |
| Exact hook/component/type signature | `search_orderly_docs` with `scope: "sdk"`                     |
| Provider, deposit, or trading flow  | `explain_workflow`                                            |
| Component pattern                   | `get_component_guide` (verify imports; examples can be stale) |
| Contract/vault addresses            | `get_contract_addresses`                                      |
| REST/WS endpoint details            | `get_api_info`                                                |
| Current version                     | `npm view @orderly.network/react-app version`                 |

## Related skills

- **orderly-onboarding** — CLI operations, broker IDs, MCP setup, and platform overview.
