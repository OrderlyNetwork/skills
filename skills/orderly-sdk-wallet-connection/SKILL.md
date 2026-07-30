---
name: orderly-sdk-wallet-connection
description: Comprehensive guide to integrating wallet connection in Orderly Network DEX applications, supporting both EVM (Ethereum, Arbitrum, etc.) and Solana wallets, plus Privy social login.
---

# Orderly Network: SDK Wallet Connection

A comprehensive guide to integrating wallet connection in Orderly Network DEX applications, supporting EVM (Ethereum, Arbitrum, Optimism, Base, etc.) and Solana wallets, with an optional Privy (social login) path.

## When to Use

- Setting up wallet connection for a new DEX
- Supporting multiple wallet types (MetaMask, Phantom, etc.)
- Implementing chain switching
- Choosing between the standard connector and Privy

## Prerequisites

- Orderly SDK packages installed (see `orderly-sdk-install-dependency`)
- Providers configured (see `orderly-sdk-dex-architecture`)
- Wallet packages installed

## Overview

Orderly Network supports **omnichain trading** — users connect wallets from multiple ecosystems:

- **EVM Chains**: Arbitrum, Optimism, Base, Ethereum, BSC, etc.
- **Solana**: Mainnet and Devnet

The SDK ships **two** connector packages; pick one:

| Package                                   | Use when                                                             |
| ----------------------------------------- | -------------------------------------------------------------------- |
| `@orderly.network/wallet-connector`       | Default — Web3-Onboard (EVM) + Solana wallet adapters                |
| `@orderly.network/wallet-connector-privy` | You want social login (Google, X, email, passkey) / embedded wallets |

The provider is selected at runtime: if a Privy app id is configured, the Privy connector is used; otherwise the standard connector.

> **Both `evmInitial` and `solanaInitial` are configured props** — they carry the wallet network and the wallet list. A real DEX does not omit them. The reference template always passes them (disabling one side individually via config flags).

## Required Dependencies

```bash
# Connector package
npm install @orderly.network/wallet-connector

# EVM wallets
npm install @web3-onboard/injected-wallets @web3-onboard/walletconnect \
            @binance/w3w-blocknative-connector wagmi

# Solana wallets
npm install @solana/wallet-adapter-base @solana/wallet-adapter-wallets \
            @solana-mobile/wallet-adapter-mobile
```

| Package                                | Purpose                                      |
| -------------------------------------- | -------------------------------------------- |
| `@web3-onboard/injected-wallets`       | MetaMask, Rabby, Coinbase, etc.              |
| `@web3-onboard/walletconnect`          | WalletConnect (mobile / multi-platform)      |
| `@binance/w3w-blocknative-connector`   | Binance Web3 Wallet                          |
| `wagmi`                                | EVM connectors (`injected`, `walletConnect`) |
| `@solana/wallet-adapter-base`          | Solana adapter base + network enum           |
| `@solana/wallet-adapter-wallets`       | Phantom, Solflare, Ledger adapters           |
| `@solana-mobile/wallet-adapter-mobile` | Solana Mobile Wallet Adapter                 |

## Standard Connector

### 1. Wallet Connector Provider

Wrap `OrderlyAppProvider` with `WalletConnectorProvider`, passing both initial configs:

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
  projectId,
  appName,
}: {
  children: ReactNode;
  networkId: NetworkId;
  disableEvm?: boolean; // from your config
  disableSolana?: boolean; // from your config
  projectId?: string; // WalletConnect project id (see below)
  appName?: string;
}) => (
  <WalletConnectorProvider
    evmInitial={disableEvm ? undefined : getEvmInitialConfig(projectId, appName)}
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

> `networkId` must be consistent between `WalletConnectorProvider` (Solana network) and `OrderlyAppProvider`. The reference provider derives both from the same `getNetworkId()`.

### 2. EVM Wallet Configuration

The template builds EVM connectors in **two** layers: `getEvmConnectors()` (wagmi `CreateConnectorFn[]`, used by the Privy path) and `getOnboardEvmWallets()` (Web3-Onboard modules, used by `getEvmInitialConfig()` for the standard connector).

```tsx
// app/utils/walletConfig.ts (excerpt)
import { CreateConnectorFn } from 'wagmi';
import { injected, walletConnect } from 'wagmi/connectors';
import injectedOnboard from '@web3-onboard/injected-wallets';
import walletConnectOnboard from '@web3-onboard/walletconnect';
import binanceWallet from '@binance/w3w-blocknative-connector';

// wagmi connectors (Privy path)
export const getEvmConnectors = (
  projectId?: string,
  appName = 'Orderly App'
): CreateConnectorFn[] => {
  const connectors: CreateConnectorFn[] = [injected()];
  if (projectId && typeof window !== 'undefined') {
    connectors.push(
      walletConnect({
        projectId,
        showQrModal: true,
        metadata: {
          name: appName,
          description: appName,
          url: window.location.origin,
          icons: [`${window.location.origin}/favicon.webp`],
        },
      })
    );
  }
  return connectors;
};

// Web3-Onboard modules (standard connector path)
export const getOnboardEvmWallets = (projectId?: string) => {
  if (!projectId || typeof window === 'undefined') return [];
  return [
    injectedOnboard(),
    binanceWallet({ options: { lng: 'en' } }),
    walletConnectOnboard({
      projectId,
      qrModalOptions: { themeMode: 'dark' },
      dappUrl: window.location.origin,
    }),
  ];
};

// evmInitial shape: { options: { wallets, appMetadata } } — undefined if no projectId
export const getEvmInitialConfig = (projectId?: string, appName = 'Orderly App') => {
  const wallets = getOnboardEvmWallets(projectId);
  return wallets.length > 0
    ? { options: { wallets, appMetadata: { name: appName, description: appName } } }
    : undefined;
};
```

> Source your **WalletConnect project id** however your app prefers (the reference template reads it from `.env` as `import.meta.env.VITE_WALLETCONNECT_PROJECT_ID`). Get one at https://cloud.walletconnect.com. Without it, only the injected browser wallet is available.

### 3. Solana Wallet Configuration

```tsx
// app/utils/walletConfig.ts (excerpt)
import {
  WalletAdapterNetwork,
  WalletError,
  WalletNotReadyError,
  Adapter,
} from '@solana/wallet-adapter-base';
import {
  LedgerWalletAdapter,
  PhantomWalletAdapter,
  SolflareWalletAdapter,
} from '@solana/wallet-adapter-wallets';
import {
  createDefaultAddressSelector,
  createDefaultAuthorizationResultCache,
  SolanaMobileWalletAdapter,
} from '@solana-mobile/wallet-adapter-mobile';
import type { NetworkId } from '@orderly.network/types';

export const getSolanaWallets = (networkId: NetworkId) => {
  if (typeof window === 'undefined') return [];
  return [
    new PhantomWalletAdapter(),
    new SolflareWalletAdapter(),
    new LedgerWalletAdapter(),
    new SolanaMobileWalletAdapter({
      addressSelector: createDefaultAddressSelector(),
      appIdentity: { uri: `${location.protocol}//${location.host}` },
      authorizationResultCache: createDefaultAuthorizationResultCache(),
      chain: networkId === 'mainnet' ? WalletAdapterNetwork.Mainnet : WalletAdapterNetwork.Devnet,
      onWalletNotFound: () => Promise.reject(new WalletNotReadyError('wallet not ready')),
    }),
  ];
};

export const getSolanaConfig = (networkId: NetworkId) => ({
  wallets: getSolanaWallets(networkId),
  onError: (error: WalletError, adapter?: Adapter) => {
    console.log(error, adapter);
  },
});
```

> `SolanaMobileWalletAdapter` (from `@solana-mobile/wallet-adapter-mobile`) is required for Solana Mobile / Saga device support. Omitting it means mobile Solana users cannot connect.

## Accessing Wallet State

Use the `useWalletConnector()` hook. Its return is **flat** (top-level `connect`, `disconnect`, `setChain`, etc.), not nested under `wallet`.

```tsx
import { useWalletConnector } from '@orderly.network/hooks';

function WalletStatus() {
  const {
    connect, // (options?) => Promise<WalletState[]>
    disconnect, // (options)  => Promise<any[]>
    connecting, // boolean — connection in progress
    wallet, // WalletState | null
    connectedChain, // ConnectedChain | null
    setChain, // ({ chainId }) => Promise<any>
    chains, // available chains
    settingChain, // boolean
    namespace, // "evm" | "solana" | null
  } = useWalletConnector();

  return (
    <div>
      {wallet ? (
        <>
          <p>Connected: {wallet.accounts[0].address}</p>
          <p>Chain: {connectedChain?.id}</p>
          <button onClick={() => disconnect({})}>Disconnect</button>
        </>
      ) : (
        <button disabled={connecting} onClick={() => connect({ chainId: 42161 })}>
          {connecting ? 'Connecting…' : 'Connect Wallet'}
        </button>
      )}
    </div>
  );
}
```

### Switch Chains

```tsx
const { setChain } = useWalletConnector();
await setChain({ chainId: 10 }); // Optimism
await setChain({ chainId: 8453 }); // Base
```

## Account State Machine

After a wallet connects, the user must complete Orderly account setup (create an account key, enable trading). Track it with `useAccount()`, whose `state.status` is an `AccountStatusEnum`.

```
NotConnected(0) → Connected(1) → NotSignedIn(2) → SignedIn(3) → EnableTrading(5)
                                     │
                                     └→ DisabledTrading(4)
EnableTradingWithoutConnected(-1) — trading enabled without an explicit wallet
```

| Status (`AccountStatusEnum`)    | Value | Meaning                                  |
| ------------------------------- | ----- | ---------------------------------------- |
| `EnableTradingWithoutConnected` | -1    | Trading enabled without wallet connected |
| `NotConnected`                  | 0     | No wallet connected                      |
| `Connected`                     | 1     | Wallet connected, no Orderly account     |
| `NotSignedIn`                   | 2     | Account exists, not signed in            |
| `SignedIn`                      | 3     | Signed in to Orderly                     |
| `DisabledTrading`               | 4     | Trading disabled                         |
| `EnableTrading`                 | 5     | Trading enabled                          |

> `AccountStatusEnum` is exported from **`@orderly.network/types`**, not `@orderly.network/hooks`.

```tsx
import { useAccount } from '@orderly.network/hooks';
import { AccountStatusEnum } from '@orderly.network/types';

function AccountStatus() {
  const { account, state, createOrderlyKey, createAccount } = useAccount();

  switch (state.status) {
    case AccountStatusEnum.NotConnected:
      return <ConnectWalletButton />;
    case AccountStatusEnum.Connected:
      return <button onClick={() => createAccount()}>Create Orderly Account</button>;
    case AccountStatusEnum.NotSignedIn:
      return <button onClick={() => createOrderlyKey()}>Enable Trading</button>;
    case AccountStatusEnum.SignedIn:
    case AccountStatusEnum.EnableTrading:
      return <TradingInterface />;
  }
}
```

## UI Components for Wallet Connection

### Connect Button / Modal

The SDK exposes a connect modal through the notification/modal system:

```tsx
import { modal } from '@orderly.network/ui';
import { WalletConnectorModalId } from '@orderly.network/ui-connector';

function ConnectButton() {
  return <button onClick={() => modal.show(WalletConnectorModalId)}>Connect Wallet</button>;
}
```

A pre-styled button is also available as `ConnectWalletButton` from `@orderly.network/ui-connector`.

### AuthGuard

Wrap content that requires an authenticated, trading-enabled account:

```tsx
import { AuthGuard } from '@orderly.network/ui-connector';

function ProtectedArea() {
  return (
    <AuthGuard fallback={<ConnectPrompt />}>
      <TradingUI />
    </AuthGuard>
  );
}
```

## Supported Chains

### EVM Mainnet

| Chain    | Chain ID |
| -------- | -------- |
| Arbitrum | 42161    |
| Optimism | 10       |
| Base     | 8453     |
| Ethereum | 1        |

### EVM Testnet

| Chain            | Chain ID |
| ---------------- | -------- |
| Arbitrum Sepolia | 421614   |
| Base Sepolia     | 84532    |

### Solana

| Network | Internal ID |
| ------- | ----------- |
| Mainnet | 1399811149  |
| Devnet  | (devnet)    |

## Chain Filtering

Restrict which chains users can switch between. `chainFilter` is keyed by network, each value an array of `{ id }`:

```tsx
<OrderlyAppProvider
  brokerId="your_broker_id"
  networkId="mainnet"
  chainFilter={{
    mainnet: [{ id: 42161 }, { id: 10 }],
    testnet: [{ id: 421614 }],
  }}
  defaultChain={{ mainnet: { id: 42161 } }}
>
```

In the reference template this is built from `VITE_ORDERLY_MAINNET_CHAINS` / `VITE_ORDERLY_TESTNET_CHAINS` / `VITE_DEFAULT_CHAIN` (see `orderly-sdk-dex-architecture`).

## Handling Chain Changes

When the user switches between a mainnet chain and a testnet chain, you must flip `networkId` and reload so the SDK rebinds to the correct environment:

```tsx
<OrderlyAppProvider
  brokerId="your_broker_id"
  networkId={networkId}
  onChainChanged={(_chainId, { isTestnet }) => {
    const current = getNetworkId();
    if ((isTestnet && current === 'mainnet') ||
        (!isTestnet && current === 'testnet')) {
      localStorage.setItem('orderly_network_id', isTestnet ? 'testnet' : 'mainnet');
      setTimeout(() => window.location.reload(), 100);
    }
  }}
>
```

## Privy Integration (Social Login)

Install the Privy connector and the Privy SDK (`@privy-io/cross-app-connect`, **not** the older `@privy-io/react-auth`):

```bash
npm install @orderly.network/wallet-connector-privy @privy-io/cross-app-connect
```

The Privy provider is **`WalletConnectorPrivyProvider`** (not `WalletConnectorPrivy`). It takes `network`, `termsOfUse`, `wagmiConfig`, `solanaConfig`, `privyConfig`, and an optional `abstractConfig`.

```tsx
// app/components/orderlyProvider/privyConnector.tsx
import { ReactNode } from 'react';
import { WalletConnectorPrivyProvider, Network } from '@orderly.network/wallet-connector-privy';
import { QueryClient } from '@tanstack/query-core';
import type { NetworkId } from '@orderly.network/types';
import { getEvmConnectors, getSolanaConfig } from '@/utils/walletConfig';

type LoginMethod = 'email' | 'passkey' | 'twitter' | 'google';

type PrivyConnectorProps = {
  children: ReactNode;
  networkId: NetworkId;
  appId: string;                                  // your Privy app id (required)
  loginMethods?: LoginMethod[];                   // default ['email']
  termsOfUse?: string;
  disableEvm?: boolean;
  disableSolana?: boolean;
  enableAbstract?: boolean;
  // plus projectId / appName if you enable EVM (see getEvmConnectors)
};

const PrivyConnector = ({
  children, networkId, appId,
  loginMethods = ['email'], termsOfUse,
  disableEvm, disableSolana, enableAbstract,
}: PrivyConnectorProps) => (
  <WalletConnectorPrivyProvider
    network={networkId === 'mainnet' ? Network.mainnet : Network.testnet}
    termsOfUse={termsOfUse}
    wagmiConfig={disableEvm ? undefined : { connectors: getEvmConnectors(/* projectId */, /* appName */) }}
    solanaConfig={disableSolana ? undefined : getSolanaConfig(networkId)}
    privyConfig={{
      config: { appearance: { showWalletLoginFirst: false }, loginMethods },
      appid: appId,
    }}
    abstractConfig={enableAbstract ? { queryClient: new QueryClient() } : undefined}
  >
    {children}
  </WalletConnectorPrivyProvider>
);

export default PrivyConnector;
```

> The provider props (`appId`, `loginMethods`, etc.) come from your config — the reference template sources them from `.env` (`VITE_PRIVY_APP_ID`, `VITE_PRIVY_LOGIN_METHODS`, …). If no Privy app id is configured, use the standard `WalletConnector` instead. Login methods default to `email`.

## Best Practices

1. **Source your WalletConnect project id from config**, never hardcode it.
2. **Derive `networkId` once** (e.g. `getNetworkId()`) and feed it to both the wallet connector and `OrderlyAppProvider`.
3. **Persist network selection** in `localStorage` (`orderly_network_id`) and reload on mainnet↔testnet switches.
4. **Include `SolanaMobileWalletAdapter`** if you support Solana Mobile devices.
5. **Select Privy vs standard connector at runtime** (by whether a Privy app id is configured), not at build time.

## Related Skills

- **orderly-sdk-dex-architecture** - Provider setup and configuration
- **orderly-sdk-install-dependency** - Installing wallet packages
- **orderly-sdk-trading-workflows** - Trading implementation
- **orderly-sdk-debugging** - Troubleshooting wallet issues
- **orderly-api-authentication** - Understanding the auth flow
