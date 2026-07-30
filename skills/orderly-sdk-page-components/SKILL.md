---
name: orderly-sdk-page-components
description: Use pre-built page components from Orderly SDK to quickly assemble complete DEX pages (TradingPage, Portfolio modules, Markets, Leaderboard, Vaults, Points, Affiliate)
---

# Orderly Network: SDK Page Components

Pre-built, full-featured page components for building a complete DEX. Each page handles responsive design (desktop/mobile), state, and UI. In the reference template every page is **lazy-loaded** from `app/pages/...` and wrapped in a shared `Scaffold` layout.

## When to Use

- Building a complete DEX quickly
- Using pre-built, production-ready pages
- Implementing standard DEX pages (trading, portfolio, markets, leaderboard)

## Prerequisites

- Orderly SDK packages installed (see `orderly-sdk-install-dependency`)
- Providers configured (see `orderly-sdk-dex-architecture`)
- React Router v7 (`react-router-dom`)

## Overview

| Component                         | Package                                | Description                                              |
| --------------------------------- | -------------------------------------- | -------------------------------------------------------- |
| `TradingPage`                     | `@orderly.network/trading`             | Full trading interface (chart, orderbook, order entry)   |
| `OverviewModule.OverviewPage`     | `@orderly.network/portfolio`           | Portfolio dashboard                                      |
| `PositionsModule.PositionsPage`   | `@orderly.network/portfolio`           | Positions + history                                      |
| `OrdersModule.OrdersPage`         | `@orderly.network/portfolio`           | Orders (open, pending, filled)                           |
| `AssetsModule.AssetsPage`         | `@orderly.network/portfolio`           | Balances, deposit/withdraw                               |
| `HistoryModule.HistoryPage`       | `@orderly.network/portfolio`           | Trade / funding / settlement history                     |
| `APIManagerModule.APIManagerPage` | `@orderly.network/portfolio`           | API key management                                       |
| `FeeTierModule.FeeTierPage`       | `@orderly.network/portfolio`           | Fee tier overview                                        |
| `SettingModule.SettingPage`       | `@orderly.network/portfolio`           | Account settings                                         |
| `MarketsHomePage`                 | `@orderly.network/markets`             | Markets listing + funding comparison                     |
| `GeneralLeaderboardWidget`        | `@orderly.network/trading-leaderboard` | Trading competition leaderboard                          |
| `PointSystemPage`                 | `@orderly.network/trading-points`      | Points/merits program                                    |
| `VaultsPage`                      | `@orderly.network/vaults`              | Vault/Earn products                                      |
| `Dashboard.DashboardPage`         | `@orderly.network/affiliate`           | Affiliate/referral dashboard (inside `ReferralProvider`) |

> **Names matter.** `GeneralLeaderboardWidget` (not `LeaderboardPage`), `PointSystemPage` (not `TradingRewardsPage`; there is no `trading-rewards` package), and `VaultsPage`. The affiliate page is `Dashboard.DashboardPage`, not a bare `Dashboard`.

## The Scaffold Wrapper Pattern

Every route layout in the reference template wraps its `<Outlet/>` in `<Scaffold>` from `@orderly.network/ui-scaffold`, passing nav/footer config (built once in `useOrderlyConfig()`) and a `routerAdapter`.

```tsx
// app/pages/perp/Layout.tsx
import { Outlet } from 'react-router-dom';
import { Scaffold } from '@orderly.network/ui-scaffold';
import { useOrderlyConfig } from '@/utils/config';
import { useNav } from '@/hooks/useNav';

export default function PerpLayout() {
  const config = useOrderlyConfig();
  const { onRouteChange } = useNav();
  return (
    <Scaffold
      mainNavProps={config.scaffold.mainNavProps}
      footerProps={config.scaffold.footerProps}
      bottomNavProps={config.scaffold.bottomNavProps}
      routerAdapter={{ onRouteChange, currentPath: '/' }}
    >
      <Outlet />
    </Scaffold>
  );
}
```

Each section (`perp`, `portfolio`, `markets`, `leaderboard`, `rewards`, `vaults`, `swap`, `points`) has its own `Layout.tsx` using this pattern.

## 1. TradingPage

Rendered by `app/pages/perp/Symbol.tsx` (route `/perp/:symbol`). `tradingViewConfig` and `sharePnLConfig` come from `useOrderlyConfig()` (which builds them from the active theme + `.env` config).

```tsx
// app/pages/perp/Symbol.tsx
import { useCallback, useState } from 'react';
import { useNavigate, useParams, useSearchParams } from 'react-router-dom';
import { API } from '@orderly.network/types';
import { TradingPage } from '@orderly.network/trading';
import { useOrderlyConfig } from '@/utils/config';

export default function PerpSymbol() {
  const params = useParams();
  const [symbol, setSymbol] = useState(params.symbol!);
  const config = useOrderlyConfig();
  const navigate = useNavigate();
  const [searchParams] = useSearchParams();

  const onSymbolChange = useCallback(
    (data: API.Symbol) => {
      setSymbol(data.symbol);
      const qs = searchParams.toString();
      navigate(`/perp/${data.symbol}${qs ? `?${qs}` : ''}`);
    },
    [navigate, searchParams]
  );

  return (
    <div className="h-full">
      <TradingPage
        symbol={symbol}
        onSymbolChange={onSymbolChange}
        tradingViewConfig={config.tradingPage.tradingViewConfig}
        sharePnLConfig={config.tradingPage.sharePnLConfig}
      />
    </div>
  );
}
```

### tradingViewConfig

```tsx
tradingViewConfig: {
  scriptSRC: string;      // /tradingview/charting_library/charting_library.js
  library_path: string;   // /tradingview/charting_library/
  customCssUrl?: string;  // /tradingview/chart.css
  colorConfig?: Partial<TradingViewColorConfig>; // only in legacy mode
}
```

Build it with `createTradingViewConfig(themeSource)` (see `orderly-sdk-theming`). TradingView library files must be placed in `public/tradingview/charting_library/`.

> Route is `/perp/:symbol`, not `/trade`. The index route `/perp` redirects to a default symbol.

## 2. Portfolio Pages

Portfolio is organized into modules, each with a `…Page` export. The template routes: `portfolio` (index), `positions`, `orders`, `assets`, `api-key`, `fee`, `history`, `setting`.

```tsx
import {
  OverviewModule, PositionsModule, OrdersModule, AssetsModule,
  HistoryModule, APIManagerModule, FeeTierModule, SettingModule,
} from '@orderly.network/portfolio';

<OverviewModule.OverviewPage />
<PositionsModule.PositionsPage />
<OrdersModule.OrdersPage sharePnLConfig={config.tradingPage.sharePnLConfig} />
<AssetsModule.AssetsPage />
<HistoryModule.HistoryPage />
<APIManagerModule.APIManagerPage />
<FeeTierModule.FeeTierPage />
<SettingModule.SettingPage />
```

Portfolio routes (lazy-loaded page components, wrapped by a shared `PortfolioLayout` using `Scaffold`):

```tsx
{
  path: 'portfolio',
  element: <PortfolioLayout />,
  children: [
    { index: true, element: <PortfolioIndex /> },        // OverviewModule.OverviewPage
    { path: 'positions', element: <PortfolioPositions /> },
    { path: 'orders', element: <PortfolioOrders /> },
    { path: 'assets', element: <PortfolioAssets /> },
    { path: 'api-key', element: <PortfolioApiKey /> },   // APIManagerModule.APIManagerPage
    { path: 'fee', element: <PortfolioFee /> },          // FeeTierModule.FeeTierPage
    { path: 'history', element: <PortfolioHistory /> },
    { path: 'setting', element: <PortfolioSetting /> },  // SettingModule.SettingPage
  ],
}
```

## 3. MarketsHomePage

```tsx
import { MarketsHomePage } from '@orderly.network/markets';

<MarketsHomePage
  comparisonProps={{
    // name + icon shown in the funding-comparison list (NOT an exchanges array)
    exchangesName: import.meta.env.VITE_ORDERLY_BROKER_NAME,
    exchangesIconSrc: '/logo-secondary.webp', // optional
  }}
  onSymbolChange={(symbol) => navigate(`/perp/${symbol.symbol}`)}
/>;
```

> `comparisonProps` takes `exchangesName` and `exchangesIconSrc` (both optional), **not** an `exchanges: string[]` array.

## 4. Leaderboard

```tsx
import { GeneralLeaderboardWidget } from '@orderly.network/trading-leaderboard';

export default function LeaderboardIndex() {
  return (
    <div className="oui-py-6 oui-px-4 lg:oui-px-6">
      <GeneralLeaderboardWidget />
    </div>
  );
}
```

## 5. Points / Rewards

```tsx
import { PointSystemPage } from '@orderly.network/trading-points';
import { RouteOption } from '@orderly.network/types';

export default function PointsIndex() {
  const navigate = useNavigate();
  const onRouteChange = (p: RouteOption) => {
    if (p.href === '/perp') navigate(`/perp/${getSymbol()}`);
  };
  return <PointSystemPage onRouteChange={onRouteChange} />;
}
```

## 6. Vaults

```tsx
import { VaultsPage as VaultsPageComponent } from '@orderly.network/vaults';
<VaultsPageComponent />;
```

## 7. Affiliate / Referral

Render `Dashboard.DashboardPage` inside `<ReferralProvider>`:

```tsx
import { Dashboard, ReferralProvider } from '@orderly.network/affiliate';

export default function AffiliateIndex() {
  return (
    <ReferralProvider>
      <Dashboard.DashboardPage />
    </ReferralProvider>
  );
}
```

## Routing

Routes are defined in `app/main.tsx` with `createBrowserRouter` and a `basename` of `import.meta.env.BASE_URL`. All pages are lazy-loaded.

```tsx
const router = createBrowserRouter(
  [
    {
      path: '/',
      element: <App />,
      errorElement: <ErrorBoundary />,
      children: [
        { index: true, element: <IndexPage /> },
        {
          path: 'perp',
          element: <PerpLayout />,
          children: [
            { index: true, element: <PerpIndex /> },
            { path: ':symbol', element: <PerpSymbol /> },
          ],
        },
        {
          path: 'portfolio',
          element: <PortfolioLayout />,
          children: [
            /* …see above… */
          ],
        },
        {
          path: 'markets',
          element: <MarketsLayout />,
          children: [{ index: true, element: <MarketsIndex /> }],
        },
        {
          path: 'leaderboard',
          element: <LeaderboardLayout />,
          children: [{ index: true, element: <LeaderboardIndex /> }],
        },
        {
          path: 'rewards',
          element: <RewardsLayout />,
          children: [
            { index: true, element: <RewardsIndex /> },
            { path: 'affiliate', element: <RewardsAffiliate /> },
          ],
        },
        {
          path: 'vaults',
          element: <VaultsLayout />,
          children: [{ index: true, element: <VaultsIndex /> }],
        },
        {
          path: 'swap',
          element: <SwapLayout />,
          children: [{ index: true, element: <SwapIndex /> }],
        },
        {
          path: 'points',
          element: <PointsLayout />,
          children: [{ index: true, element: <PointsIndex /> }],
        },
      ],
    },
  ],
  { basename: basePath }
);
```

## Responsive Design

Page components handle desktop/mobile internally. Use `useScreen()` (from `@orderly.network/ui`) for your own layout decisions:

```tsx
import { useScreen } from '@orderly.network/ui';
const { isMobile } = useScreen();
```

## Best Practices

1. **Lazy-load every page** (the template imports `app/pages/...` lazily in `main.tsx`).
2. **Wrap routes in `Scaffold`** so nav/footer/account menu are consistent.
3. **Persist the symbol** (the template calls `updateSymbol(symbol)` on change) so deep links and the index route restore it.
4. **Drive `tradingViewConfig` / `sharePnLConfig` from `useOrderlyConfig()`** so they follow the active theme.
5. **Use `onSymbolChange` to update the URL** for bookmarkable trading pages.

## Related Skills

- **orderly-sdk-dex-architecture** - Project structure, providers, routing entry point
- **orderly-sdk-theming** - Theme config behind `tradingViewConfig`
- **orderly-sdk-trading-workflows** - Trading functionality details
- **orderly-ui-components** - Granular widgets vs. these high-level pages
