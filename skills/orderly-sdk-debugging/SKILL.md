---
name: orderly-sdk-debugging
description: Debug and troubleshoot common issues with the Orderly SDK including errors, WebSocket issues, authentication problems, and trading failures.
---

# Orderly Network: SDK Debugging

A guide to debugging **runtime behavior of the Orderly SDK** — account/auth state, data streams, order and deposit/withdraw failures, and the errors the SDK throws.

> This skill covers the SDK's own runtime surface. For **build/setup** problems (Vite polyfills, CSS imports, package versions) see **orderly-sdk-install-dependency** and **orderly-sdk-dex-architecture**; for **wiring** (provider props, `.env` config) see **orderly-sdk-dex-architecture**.

## When to Use

- Debugging account / authentication state ("stuck connecting", "not signed in")
- Investigating data not loading (orderbook, positions, balances)
- Troubleshooting order submission / rejections
- Troubleshooting deposit / withdrawal failures
- Understanding SDK-thrown errors (`ApiError` / `SDKError`)

## Prerequisites

- Orderly SDK providers mounted (`OrderlyAppProvider` from `@orderly.network/react-app`)
- Browser DevTools familiarity

## 1. Account & Authentication State

The SDK account moves through a fixed ladder of states. Most "nothing works" bugs are an account stuck before `SignedIn`.

```tsx
import { useAccount } from '@orderly.network/hooks';
import { AccountStatusEnum } from '@orderly.network/types';

function AccountDebugger() {
  const { state, account } = useAccount();

  // state.status is an AccountStatusEnum (numeric)
  console.log('account state', state.status, AccountStatusEnum[state.status]);
  console.log('accountId', account?.accountId);
}
```

### `AccountStatusEnum` (from `@orderly.network/types`)

| Value | Name                            | Meaning                                                       |
| ----- | ------------------------------- | ------------------------------------------------------------- |
| `-1`  | `EnableTradingWithoutConnected` | Trading enabled without a connected wallet                    |
| `0`   | `NotConnected`                  | No wallet connected                                           |
| `1`   | `Connected`                     | Wallet connected, Orderly account not yet resolved            |
| `2`   | `NotSignedIn`                   | Need an EIP-712 / Ed25519 signature to create the Orderly key |
| `3`   | `SignedIn`                      | Fully authenticated — private streams/orders work             |
| `4`   | `DisabledTrading`               | Trading disabled for this account                             |
| `5`   | `EnableTrading`                 | Trading enabled                                               |

### The authentication ladder & common stuck points

```
NotConnected ──connect wallet──▶ Connected ──resolve account──▶ NotSignedIn ──sign──▶ SignedIn
```

| Symptom                                | Likely state              | Fix                                                        |
| -------------------------------------- | ------------------------- | ---------------------------------------------------------- |
| Private data (positions/balance) empty | `< SignedIn`              | Drive the user through sign-in (signature)                 |
| App keeps prompting to connect         | `NotConnected`            | Check `WalletConnectorProvider` is mounted & configured    |
| "Sign" never completes                 | `NotSignedIn`             | Wallet rejected; re-prompt. Key expires after 365 days     |
| Orders rejected as unauthorized        | `NotSignedIn`/`Connected` | Account must be `SignedIn` to submit orders                |
| `state.validating` stays `true`        | any                       | Network/request to resolve account hung — check WS/network |

> `useAccount()` returns `{ account, state, ... }`; `state` is an `AccountState` with `status: AccountStatusEnum` and `validating: boolean`.

## 2. Data Not Loading

Public streams (orderbook, trades, mark price) work without auth. **Private** streams (positions, orders, balance, wallet) require the account to be `SignedIn`. If public data shows but private data is empty, check the account state first (§1).

### WebSocket connection health

```tsx
import { useWsStatus, WsNetworkStatus } from '@orderly.network/hooks';

function StreamHealth() {
  const ws = useWsStatus();
  // WsNetworkStatus is string-valued: 'connected' | 'unstable' | 'disconnected'
  return <span>{ws}</span>;
}
```

| `WsNetworkStatus` | Meaning                                       |
| ----------------- | --------------------------------------------- |
| `Connected`       | WS up; streams flowing                        |
| `Unstable`        | Dropped, SDK is reconnecting                  |
| `Disconnected`    | Lost and not reconnecting — re-mount provider |

### Boot / preload status

```tsx
import { usePreLoadData } from '@orderly.network/hooks';

const { done, error } = usePreLoadData();
// done === false for the lifetime of the app = the initial data fetch failed/stalled
```

### Checklist: data not loading

- [ ] `useWsStatus()` is `Connected`?
- [ ] For **private** data: `state.status >= SignedIn`?
- [ ] `networkId` matches the chains you expect (`mainnet` vs `testnet`)?
- [ ] `usePreLoadData().done === true`?
- [ ] Symbol string is exact, e.g. `PERP_ETH_USDC`?
- [ ] No SDK errors in the console (§5)?

## 3. Order Submission Errors

`useOrderEntry(symbol, options)` exposes order readiness and validation **before** you submit. Check `metaState` and call `helper.validate()`.

```tsx
import { useOrderEntry } from '@orderly.network/hooks';

function OrderDebugger() {
  const { formattedOrder, metaState, helper, maxQty, estLiqPrice, submit } =
    useOrderEntry('PERP_ETH_USDC');

  // metaState.errors: OrderValidationResult | null  (null = valid)
  // metaState.validated: boolean
  console.log({ ready: metaState.validated && !metaState.errors, maxQty, estLiqPrice });

  async function place() {
    try {
      const res = await submit({ resetOnSuccess: true });
      console.log('ok', res);
    } catch (e) {
      // SDK wraps REST failures in ApiError (see §5)
      console.error('order failed', e);
    }
  }
}
```

> Field keys you set via `setValue()` are: `side`, `order_type`, `order_price`, `order_quantity`, `reduce_only`, `tp_trigger_price`, `sl_price`. The submit mutation flag is `metaState`-driven / `isMutating` on the hook — there is **no** `isSubmitting`.

### Reading order states in the order stream

```tsx
import { useOrderStream } from '@orderly.network/hooks';
import { OrderStatus } from '@orderly.network/types';

const [orders] = useOrderStream({ status: OrderStatus.INCOMPLETE });
// OrderStatus: OPEN | NEW | FILLED | PARTIAL_FILLED | CANCELLED | REPLACED | COMPLETED | INCOMPLETE | REJECTED
```

Use `OrderStatus.INCOMPLETE` (not the string `'OPEN'`) to query open orders. A `REJECTED` entry in the stream means the server rejected a submitted order — inspect its reason field.

### Checklist: order not submitting / rejected

- [ ] `state.status` is `SignedIn`?
- [ ] Symbol exact (`PERP_ETH_USDC`)?
- [ ] `metaState.errors` is `null` (validated)?
- [ ] Quantity ≥ symbol minimum (check `maxQty`)?
- [ ] Limit price within allowed range of mark?
- [ ] Enough free collateral (`useCollateral().freeCollateral`)?

## 4. Deposit & Withdrawal Errors

### Deposit (requires ERC-20 allowance first)

```tsx
import { useDeposit } from '@orderly.network/hooks';

function DepositDebugger() {
  const { balance, allowance, approve, deposit, dst } = useDeposit({
    /* srcChainId, token, ... */
  });

  async function run(amount: string) {
    // allowance must cover amount, else approve first
    if (Number(amount) > Number(allowance)) {
      await approve(amount); // wallet prompt
    }
    await deposit(); // NOTE: deposit() takes NO arguments
  }
}
```

- `approve(amount?)` and `deposit()` both trigger wallet prompts; wrap in try/catch.
- `deposit()` takes **no arguments** — the amount comes from the hook's `quantity`/`setQuantity` state.
- `depositFee` is a `bigint`.

### Withdrawal

```tsx
import { useWithdraw } from '@orderly.network/hooks';

const { withdraw, maxAmount, unsettledPnL } = useWithdraw({
  /* srcChainId, token, decimals */
});

await withdraw({
  chainId,
  token,
  amount,
  allowCrossChainWithdraw: true,
  // receiver?: '<optional custom address>'
});
```

- Use `maxAmount` for the cap (not the deprecated `availableWithdraw`).
- `unsettledPnL` reduces what you can withdraw — if withdrawals fail as insufficient, check it.

### Reading withdrawal readiness

`WithdrawStatus` (`@orderly.network/types`): `NotSupported` | `NotConnected` | `Unsettle` (unsettled PnL blocks) | `InsufficientBalance` | `Normal`. Only `Normal` is withdrawable.

### Detecting user-rejected / insufficient transactions

These failures come from the wallet/chain, not the Orderly API. Match on `error.message`:

```ts
catch (e) {
  const msg = String((e as Error)?.message ?? '');
  if (/reject|denied/i.test(msg))  /* user rejected the wallet prompt */;
  if (/insufficient/i.test(msg))   /* not enough gas/balance on-chain */;
}
```

## 5. Reading SDK Errors

The SDK surfaces two error classes from `@orderly.network/types`:

- **`ApiError`** — REST API failures. Constructed with `(message: string, code: number)`. The numeric `code` identifies the failure class; read `error.message` for detail.
- **`SDKError`** — client-side / SDK-internal failures.

```ts
import { ApiError, SDKError } from '@orderly.network/types';

try {
  await someSdkAction();
} catch (e) {
  if (e instanceof ApiError) {
    console.error('API error', e.message /* numeric code lives on the instance */);
  } else if (e instanceof SDKError) {
    console.error('SDK error', e.message);
  }
}
```

> Don't hard-code REST error code tables — codes can change. Log `error.message`, and branch on the codes you actually observe for your flow.

### Rate limiting

REST `-1003`-style rate limits surface as `ApiError`. Retry with backoff:

```ts
async function withRetry<T>(fn: () => Promise<T>, retries = 3, base = 1000): Promise<T> {
  for (let i = 0; ; i++) {
    try {
      return await fn();
    } catch (e) {
      const code = (e as ApiError & { code?: number })?.code;
      if (code === -1003 && i < retries) {
        await new Promise((r) => setTimeout(r, base * 2 ** i));
        continue;
      }
      throw e;
    }
  }
}
```

## 5. Inspecting Hook State

A tiny logger helps track async hook output across renders:

```tsx
function useDebugHook<T>(name: string, value: T): T {
  useEffect(() => {
    console.log(`[${name}]`, value);
  }, [name, value]);
  return value;
}
```

### Remember the return shapes (common confusion)

| Hook                | Shape                                                                |
| ------------------- | -------------------------------------------------------------------- |
| `usePositionStream` | 3-tuple: `[positions, calc, { isLoading }]`                          |
| `useCollateral`     | object: `{ totalCollateral, freeCollateral, availableBalance, ... }` |
| `useOrderEntry`     | object: `{ submit, metaState, helper, setValue, ... }`               |
| `useAccount`        | object: `{ account, state, ... }` (`state.status`)                   |
| `useOrderStream`    | 2-tuple: `[orders, { total, isLoading, cancelOrder, ... }]`          |

Destructuring the wrong arity is a frequent silent bug (e.g. reading `isLoading` off the first element).

### Dev panel

```tsx
function OrderlyDebugPanel() {
  const { state } = useAccount();
  const ws = useWsStatus();
  // mount this component only while developing (gate it however your app prefers)
  return (
    <div style={{ position: 'fixed', right: 8, bottom: 8, fontSize: 11 }}>
      <div>account: {state.status}</div>
      <div>ws: {ws}</div>
    </div>
  );
}
```

## 6. Error Boundary

For **component** errors, use the SDK's `ErrorBoundary` (`@orderly.network/react-app`):

```tsx
import { ErrorBoundary } from '@orderly.network/react-app';

<ErrorBoundary fallback={<SomethingWentWrong />}>{children}</ErrorBoundary>;
```

Route-level errors (lazy-import failures, thrown loaders/actions) should be caught by your router's `errorElement`. `ErrorBoundary` is for the React tree.

## Debugging Checklist

### Nothing works / blank app

- [ ] `OrderlyAppProvider` mounted with valid `brokerId` / `networkId`?
- [ ] `usePreLoadData().done === true`?
- [ ] `useWsStatus() === Connected`?

### Private data empty

- [ ] `state.status` is `SignedIn`?
- [ ] Correct `networkId` for the chains shown?
- [ ] WS `Connected`?

### Order failing

- [ ] `SignedIn`?
- [ ] `metaState.errors === null`?
- [ ] Quantity ≥ min, price in range?
- [ ] Enough `freeCollateral`?

### Deposit / withdraw failing

- [ ] Allowance covers deposit amount (else `approve`)?
- [ ] `withdraw` amount ≤ `maxAmount`?
- [ ] `unsettledPnL` accounted for?
- [ ] Wallet prompt not rejected?

## Related Skills

- **orderly-sdk-dex-architecture** — provider wiring & config
- **orderly-sdk-wallet-connection** — wallet integration
- **orderly-api-authentication** — the sign-in / key flow
- **orderly-sdk-install-dependency** — package installation & build setup
