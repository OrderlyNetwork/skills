---
name: orderly-sdk-theming
description: Customize the visual appearance of your Orderly DEX with CSS variables, multi-theme support, logos, TradingView chart styling, and PnL share posters.
---

# Orderly Network: SDK Theming

Customize the visual appearance of your Orderly DEX. The SDK is themed by passing one or more **themes** to `OrderlyAppProvider`'s `themes` prop. Each theme bundles a set of CSS variables (`--oui-*`) and optional TradingView chart colors.

## When to Use

- Branding a DEX (colors, logo, favicon)
- Offering multiple themes (dark + light, or brand variants)
- Styling the TradingView chart to match
- Customizing PnL-share posters

## Prerequisites

- `@orderly.network/react-app`, `@orderly.network/ui` installed
- `OrderlyAppProvider` mounted

## How Theming Works

1. Define an array of **theme objects** (plain TS — see below).
2. Pass the array to `<OrderlyAppProvider themes={themes}>`.
3. The provider applies the default theme's CSS variables and exposes a theme switcher.

There is no SDK coupling to any particular config source: themes are just data. Define them inline, import them from a module, or load them from config — whatever your app prefers.

## Defining Themes

A theme is a plain object. Define 1–6 of them, with **exactly one** marked `isDefault`:

```ts
import type { ThemeCssVars } from '@orderly.network/ui';

const themes = [
  {
    id: '11111111-1111-4111-8111-111111111111', // unique UUID
    displayName: 'Dark', // shown in the theme switcher
    mode: 'dark', // "dark" | "light"
    isDefault: true, // exactly ONE theme sets this
    cssVars: {
      // optional: override --oui-* vars
      '--oui-color-primary': '#29dfa9',
      '--oui-color-base-1': '#0e0e10',
      '--oui-color-base-2': '#161618',
      '--oui-color-trading-profit': '#29dfa9',
      '--oui-color-trading-loss': '#f5618b',
      '--oui-gradient-brand-start': '#29dfa9',
      '--oui-gradient-brand-end': '#1aa37c',
    },
    tradingViewColorConfig: {
      // optional: chart colors
      chartBG: '#0e0e10',
      upColor: '#29dfa9',
      downColor: '#f5618b',
      pnlUpColor: '#29dfa9',
      pnlDownColor: '#f5618b',
      pnlZeroColor: '#8b8b8b',
      textColor: '#b8b8b8',
      qtyTextColor: '#8b8b8b',
      volumeUpColor: '#29dfa9',
      volumeDownColor: '#f5618b',
      closeIconColor: '#b8b8b8',
    },
  },
  {
    id: '22222222-2222-4222-8222-222222222222',
    displayName: 'Light',
    mode: 'light',
    cssVars: { '--oui-color-primary': '#1aa37c', '--oui-color-base-1': '#ffffff' },
  },
];
```

### Wiring into the provider

```tsx
import { OrderlyAppProvider } from '@orderly.network/react-app';

function Provider({ children }) {
  return (
    <OrderlyAppProvider brokerId={brokerId} networkId="mainnet" themes={themes}>
      {children}
    </OrderlyAppProvider>
  );
}
```

## Theme object schema

This is the shape `themes` expects. The reference template validates it with `zod`, but you can equally use it as a plain TS type.

| Field                    | Type                              | Required | Notes                                                  |
| ------------------------ | --------------------------------- | -------- | ------------------------------------------------------ |
| `id`                     | UUID string                       | yes      | Must be unique across themes                           |
| `displayName`            | string (1–50 chars)               | yes      | Shown in the theme switcher; case-insensitively unique |
| `mode`                   | `"dark"` \| `"light"`             | yes      | Base mode                                              |
| `cssVars`                | `Partial<ThemeCssVars>`           | no       | Override `--oui-*` variables (see list below)          |
| `tradingViewColorConfig` | `Partial<TradingViewColorConfig>` | no       | Chart colors (see below)                               |
| `isDefault`              | boolean                           | no       | **Exactly one** theme must set this                    |

Constraints: 1–6 themes; unique `id`s; unique `displayName`s (case-insensitive); exactly one default.

```ts
export type DexThemeConfig = Array<{
  id: string;
  displayName: string;
  mode: 'dark' | 'light';
  cssVars?: Partial<ThemeCssVars>;
  tradingViewColorConfig?: TradingViewColorConfig;
  isDefault?: boolean;
}>;
```

## CSS Variables (`--oui-*`)

Override any of these under `cssVars`. All are optional — omitting one keeps the SDK default for the theme `mode`.

**Brand / accent:** `--oui-color-primary`, `--oui-color-primary-light`, `--oui-color-primary-darken`, `--oui-color-primary-contrast`, `--oui-color-link`, `--oui-color-link-light`, `--oui-color-secondary`, `--oui-color-tertiary`, `--oui-color-quaternary`

**Semantic:** `--oui-color-danger` (+ `-light`, `-darken`, `-contrast`), `--oui-color-success` (+ …), `--oui-color-warning` (+ …)

**Surfaces / text:** `--oui-color-fill`, `--oui-color-fill-active`, `--oui-color-base-1` … `--oui-color-base-10`, `--oui-color-base-foreground`, `--oui-color-base-static`, `--oui-color-base-static-contrast`, `--oui-color-line`

**Trading PnL:** `--oui-color-trading-profit`, `--oui-color-trading-profit-contrast`, `--oui-color-trading-loss`, `--oui-color-trading-loss-contrast`

**Gradients:** `--oui-gradient-primary-{start,end}`, `--oui-gradient-secondary-*`, `--oui-gradient-success-*`, `--oui-gradient-danger-*`, `--oui-gradient-brand-{start,end,stop-start,stop-end,angle}`, `--oui-gradient-warning-*`, `--oui-gradient-neutral-*`

**Shape & spacing:** `--oui-rounded-sm`, `--oui-rounded`, `--oui-rounded-{md,lg,xl,2xl,full}`, `--oui-spacing-{xs,sm,md,lg,xl}`, `--oui-font-family`

> The full key set is `typeof DARK_THEME_CSS_VARS` (exported from `@orderly.network/ui` as `ThemeCssVars`). Any subset is valid.

## TradingView Chart Styling

Chart colors live inside a theme's `tradingViewColorConfig`. The chart config itself (library paths) is passed to `TradingPage` via `tradingViewConfig`.

### TradingViewColorConfig fields

| Field             | Type            | Notes                           |
| ----------------- | --------------- | ------------------------------- |
| `chartBG`         | hex             | Chart background                |
| `upColor`         | hex             | Up candle / line                |
| `downColor`       | hex             | Down candle / line              |
| `pnlUpColor`      | hex             | PnL positive                    |
| `pnlDownColor`    | hex             | PnL negative                    |
| `pnlZeroColor`    | hex             | PnL zero                        |
| `textColor`       | hex             | Axis / label text               |
| `qtyTextColor`    | hex             | Quantity text                   |
| `volumeUpColor`   | hex             | Volume bar (up)                 |
| `volumeDownColor` | hex             | Volume bar (down)               |
| `closeIconColor`  | hex **or** rgba | Close button icon (allows rgba) |

```tsx
<TradingPage
  symbol={symbol}
  tradingViewConfig={{
    scriptSRC: '/tradingview/charting_library/charting_library.js',
    library_path: '/tradingview/charting_library/',
    customCssUrl: '/tradingview/chart.css',
    // colorConfig is omitted when themes carry tradingViewColorConfig;
    // the chart then reads colors from the active theme.
  }}
/>
```

## Logos & Favicon

Logos are passed to `OrderlyAppProvider` via the `appIcons` prop, typed as `AppLogos` (`@orderly.network/react-app`):

```tsx
import type { AppLogos } from '@orderly.network/react-app';

const appIcons: AppLogos = {
  main: hasPrimaryLogo
    ? { component: <img src="/logo.webp" alt="logo" style={{ height: '42px' }} /> }
    : { img: '/orderly-logo.svg' }, // { img } for a plain URL
  secondary: { img: hasSecondaryLogo ? '/logo-secondary.webp' : '/orderly-logo-secondary.svg' },
};

<OrderlyAppProvider appIcons={appIcons} /* … */ />;
```

Place assets in `public/` (e.g. `public/logo.webp`). Use a base-path wrapper so they resolve under a sub-path deploy. Update `public/favicon.webp` and the `<link>` tags in `index.html` for the favicon.

## PnL Share Posters

`TradingPage` accepts a `sharePnLConfig`:

```tsx
const sharePnLConfig = {
  backgroundImages: posterBackgrounds, // URLs to your poster backgrounds
  color: 'rgba(255,255,255,0.98)',
  profitColor: 'rgba(41,223,169,1)',
  lossColor: 'rgba(245,97,139,1)',
  brandColor: 'rgba(255,255,255,0.98)',
  refLink: window.location.origin,
  refSlogan: brokerName, // your brand slogan
};

<TradingPage sharePnLConfig={sharePnLConfig} /* … */ />;
```

## Tailwind

The SDK does **not** ship a Tailwind preset. Point the content glob at your source and do **not** define brand colors in Tailwind — use the `--oui-*` CSS variables instead.

```ts
// tailwind.config.ts
import type { Config } from 'tailwindcss';
export default {
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
/* global styles (imported once at app entry) */
@import '@orderly.network/ui/dist/styles.css';
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Target SDK elements via their `oui-*` class names and the CSS variables for custom overrides:

```css
.oui-card {
  border-radius: var(--oui-rounded-lg);
}
```

## Loading themes from config (optional)

If you want to swap themes without a code change, store the theme array as JSON and load it at runtime. The reference template does this with a Vite `.env` var and a small validating loader:

```bash
# .env  (single line — shown pretty-printed for readability)
VITE_ORDERLY_THEME_CONFIG=[{"id":"…","displayName":"Dark","mode":"dark","isDefault":true,"cssVars":{...}}]
```

```ts
// resolves + zod-validates the JSON into a DexThemeConfig[]; falls back to a
// single synthesized default theme when the var is absent/invalid
const { themes } = resolveDexThemeConfig(import.meta.env.VITE_ORDERLY_THEME_CONFIG);
```

> A runtime-config indirection (`public/config.js` / `window.__RUNTIME_CONFIG__`) is only useful for multi-instance deployments where one build is reused with different themes — an **Orderly One** concern, documented in `orderly-one-dex`. For a normal DEX, a static theme array (or `.env`) is sufficient.

## Related Skills

- **orderly-sdk-dex-architecture** - Provider setup, where `themes` is wired in
- **orderly-ui-components** - Component library that consumes these CSS variables
- **orderly-sdk-install-dependency** - Installing the UI packages
