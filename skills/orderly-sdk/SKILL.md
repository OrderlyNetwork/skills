---
name: orderly-sdk
description: Build an Orderly perpetuals DEX with the React SDK - template-first setup, packages and providers, wallet connection, UI components, hooks, theming, and debugging.
---

# Orderly React SDK

Orderly's React SDK provides the trading UI, wallet connection, data streams, and order signing for a perpetuals DEX. Start from the template for a working DEX, or from the packages when you want to compose the app yourself.

## Two ways to start

Both are valid. The template is the default because it ships a working DEX — trading page, portfolio, markets, wallet connection, theming, and deployment config — with the wiring already done. Building from scratch with the packages gives you full control over layout, routing, and data, and the provider, package, wallet, and theming guidance below applies to both paths.

### Fork the template

Fork **[dex-template](https://github.com/OrderlyNetwork/dex-template)**.

1. Fork or clone the repo.
2. Fill `.env`: `VITE_ORDERLY_BROKER_ID`, `VITE_ORDERLY_BROKER_NAME`, `VITE_DEFAULT_CHAIN`, `VITE_ORDERLY_MAINNET_CHAINS`, `VITE_ORDERLY_TESTNET_CHAINS`, `VITE_WALLETCONNECT_PROJECT_ID`.
3. Set the theme (see **Theming**).
4. Deploy, then check the broker with `orderly broker-status --address 0x…`. `orderly` is the Orderly trading CLI; install it from [orderly.network/cli.sh](https://orderly.network/cli.sh).

### Build from scratch

Create a React app, install the packages below, mount the providers, and compose the pages or widgets you need. Nothing about starting from scratch requires a different setup; it only means you write the routing, layout, and page composition yourself.

## Versions

The current line is 3.x. Keep every `@orderly.network/*` package on the same version; `@orderly.network/trading-points` is the exception and follows its own line. Check the installed version with `npm view @orderly.network/react-app version` or MCP `search_orderly_docs` instead of pinning a patch here.

`@orderly.network/hooks` comes in through `react-app`; install it separately only for a hooks-only app.

## Install

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

For social login, replace the standard connector:

```bash
npm install @orderly.network/wallet-connector-privy @privy-io/cross-app-connect
```

## Packages and imports

Each component is exported from the package that owns it:

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

Orderbook components are in `@orderly.network/trading`; the chart is `TradingView` from `@orderly.network/ui`.

## Providers

Nest them in this order: `OrderlyLocaleProvider` → wallet connector → `OrderlyAppProvider` → app. The wallet connector wraps `OrderlyAppProvider`, not the other way around.

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

`OrderlyAppProvider` mounts `QueryClientProvider`, `ModalProvider`, `TooltipProvider`, `AppConfigProvider`, `OrderlyThemeProvider`, `OrderlyConfigProvider`, and `AppStateProvider` internally. Adding any of them again breaks context.

`TradingPage` owns symbol state and needs no extra provider. `chainFilter` takes `{ mainnet?: [{ id }], testnet?: [{ id }] }`. `defaultChain` takes `{ mainnet: { id } }`.

## Wallets

The standard connector needs configured `evmInitial` and `solanaInitial` props; both are required in a real deployment. Put the configuration in `walletConfig.ts` (the template has one) and keep the Solana network aligned with `OrderlyAppProvider.networkId`.

`useWalletConnector()` returns a flat object: `connect`, `disconnect`, `connecting`, `wallet`, `connectedChain`, `setChain`, `namespace`. All fields are top-level.

For social login, use `WalletConnectorPrivyProvider` from `@orderly.network/wallet-connector-privy` and set `VITE_PRIVY_APP_ID`. Provider props and per-wallet setup: MCP `search_orderly_docs` with `wallet connector`.

## Pages and components

Prefer the page components over hand-built widget layouts. The template composes them already; when building from scratch, mount the pages you need on your own routes.

```tsx
import { TradingPage } from '@orderly.network/trading';

<TradingPage symbol="PERP_ETH_USDC" tradingViewConfig={tradingViewConfig} />;
```

- `Scaffold` (`@orderly.network/ui-scaffold`) provides nav, account menu, and layout.
- Portfolio modules: `OverviewModule`, `PositionsModule`, `OrdersModule`, `AssetsModule`, `HistoryModule`, `FeeTierModule`, `APIManagerModule`.
- Markets: `MarketsHomePage`. Vaults: `VaultsPage` from `@orderly.network/vaults`.
- Leaderboard: `GeneralLeaderboardWidget` from `@orderly.network/trading-leaderboard`. Points: `PointSystemPage` from `@orderly.network/trading-points`. Affiliate: `Dashboard.DashboardPage` inside `ReferralProvider` from `@orderly.network/affiliate`.

The template's `app/pages/` shows the routing and prop wiring. For anything not listed here, search the MCP docs.

## Hooks

Use the SDK hooks rather than reimplementing order or position state.

| Area           | Hooks                                                                     |
| -------------- | ------------------------------------------------------------------------- |
| Account / auth | `useAccount`, `useWalletConnector`                                        |
| Orders         | `useOrderEntry`, `useOrderStream`                                         |
| Positions      | `usePositionStream`, `useTPSLOrder`                                       |
| Market data    | `useOrderbookStream`, `useMarkPrice`, `useTickerStream`, `useSymbolsInfo` |
| Balances       | `useCollateral`, `useHoldingStream`                                       |
| Funds          | `useDeposit`, `useWithdraw`                                               |
| Risk           | `useLeverage`, `useMarginRatio`, `useChains`                              |

Three return shapes cause most mistakes:

- `useAccount()` returns `{ account, state }`; `state.status` is a numeric `AccountStatusEnum` from `@orderly.network/types`, and private data or orders require `SignedIn`.
- `useOrderEntry(symbol)` returns `formattedOrder`, `metaState`, `helper`, `maxQty`, `estLiqPrice`, `submit`. `metaState.errors === null` means the order validates; `isMutating` reports an in-flight submission.
- `useOrderStream({ status: OrderStatus.INCOMPLETE })` takes the enum, not the string `'OPEN'`.

Full signatures: MCP resource `orderly://sdk/hooks` or `search_orderly_docs` with `scope: "sdk"`.

## Theming

Pass 1–6 theme objects to `OrderlyAppProvider`'s `themes` prop. Exactly one sets `isDefault`.

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

Variables use the `--oui-*` prefix (v2.8+; v1 used `--orderly`). Any subset works. Chart colors live in `tradingViewColorConfig` on the theme; library paths live in `tradingViewConfig` on `TradingPage`. Logos go through `appIcons`.

Full variable list and PnL share posters: MCP `explain_workflow("theming")`.

## Build setup

- **Tailwind:** point `content` at `app/**/*.{ts,tsx}` and import `@orderly.network/ui/dist/styles.css` in the CSS entry. Feature packages reuse those base styles, so one stylesheet covers the UI.
- **Vite:** polyfill only `buffer`, `crypto`, and `stream`. Adding `util` or `globals` breaks the build. Use `vite-plugin-cjs-interop` for CommonJS dependencies and `vite-tsconfig-paths` for aliases.
- **TradingView:** charts need the TradingView library files served by your app. Copy the template's chart setup; the API does not ship them.
- **React:** 18+.

## Debugging

| Symptom                         | Check                                                                                                    |
| ------------------------------- | -------------------------------------------------------------------------------------------------------- |
| Blank app                       | `OrderlyAppProvider` mounted with valid `brokerId`/`networkId`; `usePreLoadData().done`; `useWsStatus()` |
| Private data empty              | `state.status === SignedIn`; correct network; WebSocket connected                                        |
| Sign-in loops                   | Wallet connector mounted and configured; user rejected the signature                                     |
| Orders rejected                 | `metaState.errors === null`; symbol exact; quantity ≥ min; price in range; free collateral               |
| Deposit fails                   | ERC-20 allowance first; correct vault address for the chain                                              |
| Withdraw fails                  | Amount ≤ `maxAmount`; unsettled PnL accounted; wallet prompt accepted                                    |
| `createClient` / `.mjs` errors  | Bundler interop and Vite polyfills (see **Build setup**)                                                 |
| Wrong imports / missing exports | The component lives in a different package; check the table above                                        |

Log `AccountStatusEnum[state.status]` when the account is stuck. For HTTP failures, inspect the SDK's `ApiError` (`status`, `code`, `message`).

## Reference

| Need                                | Call                                                          |
| ----------------------------------- | ------------------------------------------------------------- |
| Exact hook/component/type signature | `search_orderly_docs` with `scope: "sdk"`                     |
| Provider, deposit, or trading flow  | `explain_workflow`                                            |
| Component pattern                   | `get_component_guide` (verify imports; examples can be stale) |
| Contract/vault addresses            | `get_contract_addresses`                                      |
| REST/WS endpoint details            | `get_api_info`                                                |
| Current version                     | `npm view @orderly.network/react-app version`                 |
