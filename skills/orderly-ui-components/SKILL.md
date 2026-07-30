---
name: orderly-ui-components
description: Build trading interfaces using pre-built React components - OrderEntry, Positions, TradingPage, Scaffold, Tables, Modals, Sheets
---

# Orderly Network: UI Components

Build trading interfaces with Orderly's pre-built React components. The SDK ships a base component library (`@orderly.network/ui`), granular trading widgets (`ui-order-entry`, `ui-positions`, …), high-level feature pages (`trading`, `portfolio`, `markets`), and an app scaffold (`ui-scaffold`).

> **Imports matter.** Components live in **specific** packages — do not import everything from `@orderly.network/react` (that package does not exist). Use `@orderly.network/react-app` for the provider, `@orderly.network/ui` for base components, and the feature package for page-level widgets. See the package table below.

## When to Use

- Rapidly building a trading UI
- Using pre-built, styled components
- Implementing standard trading interface patterns

## Prerequisites

- React 18+
- `@orderly.network/react-app`, `@orderly.network/ui` installed
- `OrderlyAppProvider` wrapping your app
- Tailwind CSS configured (content glob on `app/`)

## Where Components Live

| Component                 | Package                           | Notes                                                   |
| ------------------------- | --------------------------------- | ------------------------------------------------------- |
| `OrderlyAppProvider`      | `@orderly.network/react-app`      | The app provider (the only export you need from here)   |
| `TradingPage`             | `@orderly.network/trading`        | Full trading page (chart + orderbook + order entry)     |
| `OrderBook`, `LastTrades` | `@orderly.network/trading`        | Orderbook + recent trades                               |
| `OrderEntryWidget`        | `@orderly.network/ui-order-entry` | Order entry form (`OrderEntry` is the headless variant) |
| `PositionsWidget`         | `@orderly.network/ui-positions`   | Positions table                                         |
| `TradingView`             | `@orderly.network/ui`             | Chart wrapper                                           |
| `Table`, `Modal`, `Sheet` | `@orderly.network/ui`             | Base primitives                                         |
| `Scaffold`                | `@orderly.network/ui-scaffold`    | App layout (nav, footer, account menu)                  |
| `AuthGuard`               | `@orderly.network/ui-connector`   | Gate content behind auth/trading                        |

> `@orderly.network/ui-orderbook` and `@orderly.network/ui-chart` do **not** exist. Orderbook lives in `@orderly.network/trading`; the chart is `TradingView` in `@orderly.network/ui`.

## Provider Hierarchy

`WalletConnectorProvider` (or Privy) **wraps** `OrderlyAppProvider`, not the other way around. There is no `TradingPageProvider` or `SymbolProvider` to add — `TradingPage` manages its own symbol context internally.

```tsx
import { OrderlyAppProvider } from '@orderly.network/react-app';
import { WalletConnectorProvider } from '@orderly.network/wallet-connector';

function App() {
  return (
    <WalletConnectorProvider evmInitial={…} solanaInitial={…}>
      <OrderlyAppProvider brokerId="your_broker_id" networkId="mainnet">
        <YourApp />
      </OrderlyAppProvider>
    </WalletConnectorProvider>
  );
}
```

> Do not add `QueryClientProvider`, `ModalProvider`, or `TooltipProvider` yourself — `OrderlyAppProvider` sets those up internally.

## Trading Page (recommended)

For a complete trading interface, use `TradingPage` — it composes the chart, orderbook, order entry, and positions for you. This is what the reference template uses; hand-assembling the widgets below is only for custom layouts.

```tsx
import { TradingPage } from '@orderly.network/trading';

function PerpSymbol({ symbol }: { symbol: string }) {
  return <TradingPage symbol={symbol} />;
}
```

`TradingPage` accepts a `tradingViewConfig` (chart), `sharePnLConfig`, and layout options. See `orderly-sdk-page-components`.

## Order Entry

`OrderEntryWidget` is the self-contained widget (pass `symbol`). `OrderEntry` is the presentational component that takes the script return values (use only for advanced headless wiring).

```tsx
import { OrderEntryWidget } from '@orderly.network/ui-order-entry';

function OrderEntryPanel({ symbol }: { symbol: string }) {
  return (
    <div className="rounded-lg p-4">
      <OrderEntryWidget symbol={symbol} />
    </div>
  );
}
```

> There is no `OrderEntryProvider`. Pass the `symbol` prop directly.

## Positions

```tsx
import { PositionsWidget } from '@orderly.network/ui-positions';

function PositionsPanel({ symbol, onSymbolChange }: {
  symbol?: string;
  onSymbolChange?: (s: API.Symbol) => void;
}) {
  return (
    <PositionsWidget
      symbol={symbol}            // optional — filter to one symbol
      onSymbolChange={onSymbolChange}
      pnlNotionalDecimalPrecision={2}
      sharePnLConfig={…}         // optional PnL-share config
    />
  );
}
```

For the full portfolio (positions + history + orders + assets), prefer `OverviewModule`/`PositionsModule` from `@orderly.network/portfolio` (see `orderly-sdk-page-components`).

## Orderbook & Trades

```tsx
import { OrderBook, LastTrades } from '@orderly.network/trading';

function MarketData({ symbol }: { symbol: string }) {
  return (
    <>
      <OrderBook symbol={symbol} level={15} />
      <LastTrades symbol={symbol} />
    </>
  );
}
```

## Chart

```tsx
import { TradingView } from '@orderly.network/ui';

function ChartPanel({ symbol }: { symbol: string }) {
  return (
    <div className="h-[500px]">
      <TradingView symbol={symbol} />
    </div>
  );
}
```

`TradingView` reads chart library paths from the `tradingViewConfig` on `TradingPage`/provider; the TradingView library must be placed in `public/tradingview/` (see `orderly-sdk-dex-architecture`).

## App Scaffold

`Scaffold` from `@orderly.network/ui-scaffold` renders the responsive layout: top nav, left sidebar (desktop), bottom nav (mobile), account menu, and footer. Pass nav props and a `routerAdapter` for navigation.

```tsx
import { Scaffold } from '@orderly.network/ui-scaffold';

function Layout({ children }: { children: React.ReactNode }) {
  return (
    <Scaffold
      mainNavProps={navConfig} // top navigation items
      bottomNavProps={bottomNavConfig} // mobile bottom nav
      footerProps={footerConfig}
      routerAdapter={{ to: (path) => navigate(path) }} // your router
    >
      {children}
    </Scaffold>
  );
}
```

## Wallet Connect (UI)

There is no standalone `WalletConnect` component export. Trigger the SDK's connect modal imperatively, or gate content with `AuthGuard`:

```tsx
import { modal } from '@orderly.network/ui';
import { WalletConnectorModalId, AuthGuard } from '@orderly.network/ui-connector';

function Header() {
  return <button onClick={() => modal.show(WalletConnectorModalId)}>Connect</button>;
}

function ProtectedOrders() {
  return (
    <AuthGuard>
      <PositionsWidget />
    </AuthGuard>
  );
}
```

## Base Components (`@orderly.network/ui`)

### Data Table

The `Table` uses a column definition API (`dataIndex`, `render`, `title`):

```tsx
import { Table, type Column } from '@orderly.network/ui';

type TradeRow = { id: string; price: number; size: number; side: 'BUY' | 'SELL' };

function TradesTable({ data }: { data: TradeRow[] }) {
  const columns: Column<TradeRow>[] = [
    {
      title: 'Price',
      dataIndex: 'price',
      render: (value, record) => (
        <span className={record.side === 'BUY' ? 'text-green-500' : 'text-red-500'}>
          {value.toFixed(2)}
        </span>
      ),
    },
    { title: 'Size', dataIndex: 'size', formatter: (v) => v.toFixed(4) },
    { title: 'Time', dataIndex: 'time' },
  ];

  return <Table columns={columns} dataSource={data} getKey={(r) => r.id} />;
}
```

`Column<RecordType>` fields include: `title`, `dataIndex`, `render`, `formatter`, `width`, `align`, `fixed`, `onSort`, `className`, `getKey`, `hint`.

### Modal / Sheet / Toast

```tsx
import { Modal, Sheet, toast } from '@orderly.network/ui';

// Imperative toast (no provider needed — OrderlyAppProvider hosts it)
toast.success('Order placed');
toast.error('Order failed');

// Declarative modal & sheet use trigger/content composition
<Modal>
  <Modal.Trigger asChild>
    <button>Open</button>
  </Modal.Trigger>
  …
</Modal>;
<Sheet>
  <Sheet.Trigger asChild>
    <button>Open</button>
  </Sheet.Trigger>
  …
</Sheet>;
```

> The exact composition sub-components (e.g. `ModalTrigger`) follow Radix-style naming; check the installed `@orderly.network/ui` types for the precise names if building declaratively. For toasts, prefer the imperative `toast` helper.

### Other primitives

`Button`, `Input`, `Select`, `Tabs`, `Spinner`, `Tooltip`, `NoData`, `Flex`, `cn`, `useScreen` — all from `@orderly.network/ui`.

## Styling

Components use Tailwind utility classes and CSS variables (themeable via `themes` prop — see `orderly-sdk-theming`). Import the base stylesheet once:

```css
/* app/styles/index.css */
@import '@orderly.network/ui/dist/styles.css';
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Common Issues

- **"Module not found: @orderly.network/react"** — Use `@orderly.network/react-app`. `react` is not a package.
- **Components not rendering** — Ensure `OrderlyAppProvider` (and a wallet connector) wraps the app; symbol-dependent widgets need a valid `symbol`.
- **Missing styles** — Import `@orderly.network/ui/dist/styles.css` and point Tailwind's `content` at `app/`.
- **"TradingPageProvider/SymbolProvider is not exported"** — These don't exist. `TradingPage` handles its own context.

## Related Skills

- **orderly-sdk-react-hooks** - Hook reference for the data behind these components
- **orderly-sdk-page-components** - High-level page composition
- **orderly-sdk-dex-architecture** - Provider setup and project structure
- **orderly-sdk-theming** - Theming via CSS variables
