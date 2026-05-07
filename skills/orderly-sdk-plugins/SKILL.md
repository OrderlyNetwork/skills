---
name: orderly-sdk-plugins
description: Build, install, and submit Orderly SDK v3 plugins/modules using orderly-devkit, interceptors, and OrderlyAppProvider plugins.
---

# Orderly Network: SDK Plugins & Module Marketplace

Use this skill when building or integrating Orderly SDK v3 modules, frontend plugins, interceptors, or Marketplace submissions.

## When to Use

- Creating a custom Orderly plugin or Module Marketplace module
- Installing an existing plugin into a DEX host
- Adding, wrapping, or replacing SDK UI through interceptors
- Preparing a plugin for Marketplace submission
- Helping an AI agent choose the correct plugin workflow and tooling

## Core Concepts

- A **module** is the complete feature a builder ships. It includes a frontend plugin and may include a builder-hosted backend service.
- A **plugin** is the SDK frontend extension registered with `OrderlyAppProvider`.
- Plugins require **Orderly SDK v3.0.0 or above**. Keep all `@orderly.network/*` packages on the same version.
- Plugins use interceptors to enhance, wrap, or replace components at SDK runtime target paths.

## Recommended Agent Workflow

For plugin authoring, use the official plugin skills and MCP tooling installed by `orderly-devkit`:

```bash
pnpm add -g @orderly.network/devkit
orderly-devkit skills install
orderly-devkit mcp install
```

The devkit installs four plugin playbooks:

| Skill                   | Purpose                                               |
| ----------------------- | ----------------------------------------------------- |
| `orderly-plugin-create` | Scaffold a plugin with `orderly-devkit create plugin` |
| `orderly-plugin-write`  | Implement interceptors, hooks, lifecycle, and UI      |
| `orderly-plugin-add`    | Wire a plugin into a DEX host                         |
| `orderly-plugin-submit` | Prepare manifest, `usagePrompt`, dry-run, and submit  |

Prefer the MCP docs tools while coding plugin internals. Start with docs search, then fetch exact components/types. Older notes may mention a hook-specific MCP lookup; use component/type docs instead when that tool is unavailable.

## Install an Existing Plugin

Install the plugin package, import its registrar, and pass it to `OrderlyAppProvider.plugins`.

```bash
npm install @orderly.network/onramper-plugin
```

```tsx
import { OrderlyAppProvider } from '@orderly.network/react-app';
import { registerOnramperPlugin } from '@orderly.network/onramper-plugin';

<OrderlyAppProvider
  brokerId="your_broker_id"
  brokerName="Your DEX"
  networkId="mainnet"
  plugins={[registerOnramperPlugin()]}
>
  <TradingPage />
</OrderlyAppProvider>;
```

When integrating into an existing host, locate the single top-level `OrderlyAppProvider` first. Add to an existing `plugins` array instead of creating a second provider.

## Create a New Plugin

Use the devkit scaffold rather than hand-rolling package structure:

```bash
orderly-devkit create plugin
```

During implementation:

- Keep hook calls inside React components or custom hooks, not directly inside interceptor factory functions.
- Use `@orderly.network/hooks`, `@orderly.network/ui`, and `@orderly.network/types` from the host-compatible SDK version.
- Choose the narrowest interceptor target that fits the feature.
- Test the plugin inside a real DEX host before publishing.

## Interceptor Pattern

Runtime targets are SDK-version-specific. Use the Module Marketplace docs, `@orderly.network/plugin-core`, or dev inspector tooling to confirm target strings for the installed SDK version.

Typical interceptor shape:

```tsx
export function registerExamplePlugin() {
  return {
    name: 'example-plugin',
    interceptors: [
      {
        target: 'some/runtime/target',
        component: (Original, props, api) => {
          return <Original {...props} />;
        },
      },
    ],
  };
}
```

Use wrapper interceptors when preserving host behavior. Replace components only when the module owns the complete UX and has tested equivalent states.

## Backend Modules

If the module needs persistent logic such as DCA, TWAP, grid bots, copy trading, or alerts, run a backend service and authenticate directly with the Orderly REST/WebSocket API. Do not expose admin keys, account secrets, or signing material in the frontend plugin.

## Marketplace Submission

Before submission:

- Ensure `README` and `usagePrompt` explain when agents/builders should use the module.
- Run the devkit dry run before a real submit.
- Ask for explicit user confirmation before any real Marketplace submission.

```bash
orderly-devkit login
orderly-devkit submit --dry-run
orderly-devkit submit
```

## Related Skills

- **orderly-sdk-install-dependency** - SDK package setup and version alignment
- **orderly-sdk-dex-architecture** - Provider hierarchy and host configuration
- **orderly-sdk-react-hooks** - Hooks available inside plugin components
- **orderly-sdk-theming** - Theme variables and UI styling
