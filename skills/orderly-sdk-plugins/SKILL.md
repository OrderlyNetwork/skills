---
name: orderly-sdk-plugins
description: Create, integrate, and debug Orderly SDK plugins, interceptors, widget plugins, page plugins, and custom trading layout plugins.
---

# Orderly Network: SDK Plugins

Use this skill when an agent needs to extend the Orderly Components SDK beyond normal page/component props: injected widgets, custom trading controls, plugin packages, or split/grid layout customization.

## When to Use

- Creating a new Orderly plugin package
- Injecting a widget into an existing SDK component
- Customizing the desktop trading layout
- Integrating `registerXxxPlugin()` into a host DEX
- Finding supported interceptor target paths

## Core Concepts

Orderly SDK plugins are npm packages that export a register function. The host calls that function and passes the result into the SDK plugin provider.

| Plugin Type | How It Integrates | Typical Use |
| ----------- | ----------------- | ----------- |
| Page | Host mounts it as a normal route | Campaigns, dashboards, custom pages |
| Widget | Intercepts a supported target path | Buttons, analytics panels, custom order-entry sections |
| Layout | Intercepts `Trading.Layout.Desktop` | Split/grid/pro trading layouts |

Only SDK components declared as injectable can be intercepted. Do not invent target names. Use the SDK inspector or AI docs exact lookup to verify target paths.

## Required Packages

```bash
npm install @orderly.network/plugin-core @orderly.network/ui @orderly.network/hooks

# For layout plugins
npm install @orderly.network/layout-core @orderly.network/layout-split @orderly.network/layout-grid
```

## Agent Workflow

1. Determine the plugin type: Page, Widget, or Layout.
2. Verify the target path before coding.
   - Prefer `orderly_docs_get_component` for exact targets such as `Trading.OrderEntry.SubmitSection`.
   - Use `orderly_docs_get_component_doc` for examples after the exact entity is found.
   - Use `orderly_docs_get_workflow` with `plugin-create` or `plugin-integration` for generated guidance.
3. Create a globally unique plugin ID.
   - IDs must match `/^[a-zA-Z][a-zA-Z0-9]*$/`.
   - No hyphens, underscores, dots, or leading digits.
4. Build the smallest verifiable behavior first.
5. Register the plugin in the host app in deterministic order.
6. Verify the target renders once and does not duplicate or remount unexpectedly.

## Widget Plugin Example

Use hooks inside a React wrapper component returned from the interceptor. Do not call hooks directly in the outer interceptor function.

```tsx
import { createInterceptor, type OrderlySDK } from "@orderly.network/plugin-core";
import { useOrderEntry } from "@orderly.network/hooks";

export function registerBuySellPlugin() {
  return (SDK: OrderlySDK) =>
    SDK.registerPlugin({
      id: "BuySellPlugin",
      name: "Buy Sell Plugin",
      version: "1.0.0",
      orderlyVersion: ">=3.0.0",
      interceptors: [
        createInterceptor("Trading.OrderEntry.SubmitSection", (Original, props) => {
          const Wrapper = () => {
            const symbol = (props as { symbol?: string }).symbol ?? "PERP_BTC_USDC";
            const { submit, setValue } = useOrderEntry(symbol);

            return (
              <div className="oui-flex oui-gap-2">
                <button onClick={() => { setValue("side", "BUY"); submit(); }}>
                  Buy
                </button>
                <button onClick={() => { setValue("side", "SELL"); submit(); }}>
                  Sell
                </button>
              </div>
            );
          };

          return <Wrapper />;
        }),
      ],
    });
}
```

## Host Integration

Wrap the trading app or relevant subtree with the plugin provider and pass invoked register functions.

```tsx
import { OrderlyPluginProvider } from "@orderly.network/plugin-core";
import { TradingPage } from "@orderly.network/trading";
import { registerBuySellPlugin } from "./plugins/buySell";

export function PerpPage() {
  return (
    <OrderlyPluginProvider plugins={[registerBuySellPlugin()]}>
      <TradingPage symbol="PERP_BTC_USDC" />
    </OrderlyPluginProvider>
  );
}
```

If the app already has a provider-level `plugins` prop or wrapper, add the register function there instead of creating nested ownership.

## Layout Plugins

Layout plugins target only the trading desktop layout today.

```tsx
import { OrderlyPluginProvider } from "@orderly.network/plugin-core";
import { registerLayoutSplitPlugin } from "@orderly.network/layout-split";
import { registerLayoutGridPlugin } from "@orderly.network/layout-grid";

<OrderlyPluginProvider plugins={[registerLayoutSplitPlugin()]}>
  <TradingPage symbol="PERP_BTC_USDC" />
</OrderlyPluginProvider>;

<OrderlyPluginProvider plugins={[registerLayoutGridPlugin({ persistLayout: true })]}>
  <TradingPage symbol="PERP_BTC_USDC" />
</OrderlyPluginProvider>;
```

Hosts can also pass layout props directly to `TradingPage` when they want ownership without plugin registration:

```tsx
<TradingPage
  symbol="PERP_BTC_USDC"
  layoutStrategy={myStrategy}
  getInitialLayout={() => createDefaultMyLayout(getTradingPanelIds())}
  storageKey="my_trading_layout"
/>
```

## Failure Recovery

| Problem | Action |
| ------- | ------ |
| Plugin is not visible | Verify provider wiring, invoked register function, and exact target spelling |
| Target does not match | Check SDK inspector or `orderly_docs_get_component`; fallback to nearest supported target |
| Hooks error | Move hook calls into a wrapper component returned by the interceptor |
| Duplicate rendering | Check provider nesting and plugin registration order |
| Layout does not apply | Confirm the host is not already passing `layoutStrategy` / `getInitialLayout` |
| Missing exact anchor | Document the unsupported target, use nearest parent/adjacent target, or fork SDK to add `injectable` |

## Guardrails

- Never log private keys, seeds, API secrets, or signed payload secrets from plugins.
- Use `Decimal` from `@orderly.network/utils` for financial amounts.
- Confirm network and chain before initiating wallet signing or trading actions.
- Prefer exact AI-docs lookups over semantic guesses for target paths, component props, hooks, and package surfaces.

## Related Skills

- **orderly-sdk-dex-architecture** - Provider hierarchy and DEX structure
- **orderly-sdk-page-components** - TradingPage and page module usage
- **orderly-sdk-react-hooks** - Hooks available inside plugin wrapper components
- **orderly-ui-components** - Base and trading UI component usage
