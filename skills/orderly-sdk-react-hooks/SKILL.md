---
name: orderly-sdk-react-hooks
description: Reference guide for using Orderly React SDK hooks - useOrderEntry, usePositionStream, useOrderbookStream, useCollateral, and more
---

# Orderly Network: SDK React Hooks Reference

Reference for the hooks exported by `@orderly.network/hooks`. All signatures below are verified against the installed type definitions (SDK `3.1.x`).

## When to Use

- Building React applications with Orderly
- Quick reference for hook signatures and return values
- Implementing custom trading/account UI on top of the SDK

## Prerequisites

- React 18+
- `@orderly.network/react-app` and `@orderly.network/hooks` installed
- `OrderlyAppProvider` (from `@orderly.network/react-app`) wrapping your app

> Hooks are provided by `@orderly.network/hooks`, but they only work inside `OrderlyAppProvider` from **`@orderly.network/react-app`** (not `@orderly.network/react`). `QueryClientProvider` is managed internally by `OrderlyAppProvider` — do not add your own.

## Setup

```tsx
import { OrderlyAppProvider } from '@orderly.network/react-app';

function App() {
  return (
    <OrderlyAppProvider
      brokerId="your_broker_id" // falls back to "demo" if unset
      networkId="mainnet"
      chainFilter={{
        // keyed by network, each value: [{ id }]
        mainnet: [{ id: 42161 }, { id: 10 }],
        testnet: [{ id: 421614 }],
      }}
      defaultChain={{ mainnet: { id: 42161 } }}
    >
      <YourApp />
    </OrderlyAppProvider>
  );
}
```

---

## Account Hooks

### useAccount

Account instance + state. `state.status` is an `AccountStatusEnum` (from `@orderly.network/types`), not a string.

```tsx
import { useAccount } from '@orderly.network/hooks';
import { AccountStatusEnum } from '@orderly.network/types';

const { account, state, createOrderlyKey, createAccount, switchAccount } = useAccount();

// state.status values (AccountStatusEnum):
//   NotConnected=0, Connected=1, NotSignedIn=2,
//   SignedIn=3, EnableTrading=5, DisabledTrading=4

// Example
function AccountInfo() {
  const { account, state, createAccount, createOrderlyKey } = useAccount();

  if (state.status < AccountStatusEnum.Connected) {
    return <ConnectWalletButton />;
  }
  if (state.status < AccountStatusEnum.NotSignedIn) {
    return <button onClick={() => createAccount()}>Create Account</button>;
  }
  if (state.status < AccountStatusEnum.EnableTrading) {
    return <button onClick={() => createOrderlyKey()}>Enable Trading</button>;
  }
  return <p>Account: {account.accountId}</p>;
}
```

### useWalletConnector

Wallet connection state and actions. The return is **flat** — `connect`, `disconnect`, `wallet`, etc. are top-level (not nested under `wallet`).

```tsx
import { useWalletConnector } from '@orderly.network/hooks';

const {
  connect, // (options?) => Promise<WalletState[]>
  disconnect, // (options)  => Promise<any[]>
  connecting, // boolean
  wallet, // WalletState | null
  connectedChain, // ConnectedChain | null
  setChain, // ({ chainId }) => Promise<any>
  chains, // available chains
  settingChain, // boolean
  namespace, // "evm" | "solana" | null
} = useWalletConnector();

// Example
function WalletButton() {
  const { connect, disconnect, connecting, wallet } = useWalletConnector();
  if (connecting) return <span>Connecting…</span>;
  if (wallet) {
    return <button onClick={() => disconnect({})}>Disconnect</button>;
  }
  return <button onClick={() => connect({})}>Connect Wallet</button>;
}
```

---

## Trading Hooks

### useOrderEntry

Create and submit orders for a symbol. The active overload is `useOrderEntry(symbol, options?)`. Set order fields with `setValue` (fields: `side`, `order_type`, `order_price`, `order_quantity`, `reduce_only`, `tp_trigger_price`, `sl_price`, etc.), then call `submit()`.

```tsx
import { useOrderEntry } from '@orderly.network/hooks';
import { OrderSide, OrderType } from '@orderly.network/types';

const {
  submit, // (opts?: { resetOnSuccess? }) => Promise<{ success, data, timestamp }>
  setValue, // (key: keyof FullOrderState, value, options?) => void
  setValues, // (Partial<FullOrderState>) => void
  setValuesRaw, // merge setter that skips calculate() (for advanced TPSL)
  reset,
  resetErrors,
  resetMetaState,
  formattedOrder, // Partial<FullOrderState> — the computed order
  maxQty,
  maxQtys, // { maxBuy, maxSell }
  estLiqPrice, // number | null
  estLeverage, // number | null
  currentPosition, // signed qty (+ long, - short)
  freeCollateral,
  symbolInfo, // API.SymbolExt
  helper: { validate, validator /* @deprecated → validate */ },
  metaState, // { dirty, submitted, validated, errors: OrderValidationResult | null }
  isMutating, // submission in progress (NOT "isSubmitting")
  markPrice,
  symbolLeverage,
} = useOrderEntry('PERP_BTC_USDC');

// Example
function OrderForm({ symbol }: { symbol: string }) {
  const { submit, setValue, helper, isMutating, metaState } = useOrderEntry(symbol);

  useEffect(() => {
    setValue('side', OrderSide.BUY);
    setValue('order_type', OrderType.LIMIT);
  }, [symbol]);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const result = await helper.validate();
    if (result) return; // validation errors are in metaState.errors
    await submit({ resetOnSuccess: true });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input placeholder="Price" onChange={(e) => setValue('order_price', e.target.value)} />
      <input placeholder="Quantity" onChange={(e) => setValue('order_quantity', e.target.value)} />
      <button type="submit" disabled={isMutating}>
        {isMutating ? 'Placing…' : 'Place Order'}
      </button>
    </form>
  );
}
```

### useOrderStream

Stream orders with cancel/update actions. Returns a readonly tuple `[orders, actions]`.

```tsx
import { useOrderStream } from '@orderly.network/hooks';
import { OrderStatus } from '@orderly.network/types';

const [
  orders,
  {
    total,
    isLoading,
    refresh,
    loadMore,
    cancelOrder, // (orderId, symbol?) => Promise<any>
    cancelAllOrders, // () => Promise<[any, any, any]>
    cancelAllPendingOrders, // (symbol?) => Promise<[any,any,any]>
    cancelAllTPSLOrders, // (symbol?) => Promise<any[]>
    cancelAlgoOrder, // (orderId, symbol?) => Promise<any>
    updateOrder, // (orderId, OrderEntity) => Promise<any>
    updateAlgoOrder,
    updateTPSLOrder,
    meta, // { total, current_page, records_per_page }
    errors,
    submitting,
  },
] = useOrderStream({
  status: OrderStatus.INCOMPLETE, // use the enum, not the string 'OPEN'
  symbol, // optional, omit for all symbols
  side,
  page,
  size,
  includes,
  excludes, // CombineOrderType[], filter by type
  dateRange: { from, to },
});

// Example
function OpenOrders({ symbol }: { symbol?: string }) {
  const [orders, { cancelOrder, cancelAllPendingOrders, isLoading }] = useOrderStream({
    status: OrderStatus.INCOMPLETE,
    symbol,
  });

  return (
    <div>
      <button onClick={() => cancelAllPendingOrders(symbol)}>Cancel All</button>
      {(orders ?? []).map((o) => (
        <div key={o.order_id}>
          {o.symbol} {o.side} {o.order_quantity} @ {o.order_price}
          <button onClick={() => cancelOrder(o.order_id, o.symbol)}>✕</button>
        </div>
      ))}
    </div>
  );
}
```

---

## Position Hooks

### usePositionStream

Stream positions with real-time PnL. Returns a **3-tuple** `[positions, calc, status]`, not a single object.

```tsx
import { usePositionStream } from '@orderly.network/hooks';

const [
  positions,   // { rows: API.PositionTPSLExt[], aggregated }
  calc,         // accessor fns for margin/pnl fields (see below)
  { isLoading },
] = usePositionStream(symbol?, options?);

// positions.aggregated: aggregate stats across all rows
// positions.rows[i]: { symbol, position_qty, average_open_price, mark_price,
//   unrealized_pnl, unrealized_pnl_roi, leverage, liq_price, mmr, imr, notional, ... }

// Example
function PositionsSummary() {
  const [positions, , { isLoading }] = usePositionStream();
  return (
    <table>
      {(positions?.rows ?? []).map((p) => (
        <tr key={p.symbol}>
          <td>{p.symbol}</td>
          <td>{p.position_qty}</td>
          <td>{p.unrealized_pnl?.toFixed(2)}</td>
        </tr>
      ))}
    </table>
  );
}
```

### useTPSLOrder

Manage Take-Profit / Stop-Loss for a position. Returns `[computed, actions]`.

```tsx
import { useTPSLOrder } from '@orderly.network/hooks';

const [computed, {
  setValue,      // (key, number|string|boolean) => void
  setValues,     // (Partial<ComputedAlgoOrder>) => void
  submit,        // (params?: { accountId? }) => Promise<any>
  validate,      // (otherErrors?) => Promise<AlgoOrderEntity>
  deleteOrder,   // (orderId, symbol) => Promise<any>
  errors,
  metaState,     // { dirty, submitted, validated, errors }
  isCreateMutating, isUpdateMutating,
}] = useTPSLOrder(position, options?);

// position: { symbol, average_open_price, position_qty, ... }
// options:  { defaultOrder?, isEditing?, positionType? }

// Example
function TPSLForm({ position }) {
  const [, { setValue, validate, submit, isCreateMutating }] = useTPSLOrder(position);
  const handleSet = async () => {
    setValue('positionTPSLOrderTPTriggerPrice', 71000);
    setValue('positionTPSLOrderSLTriggerPrice', 69000);
    await validate();
    await submit();
  };
  return <button onClick={handleSet} disabled={isCreateMutating}>Set TP/SL</button>;
}
```

---

## Market Data Hooks

### useOrderbookStream

Real-time orderbook.

```tsx
import { useOrderbookStream } from '@orderly.network/hooks';

const { asks, bids, isLoading } = useOrderbookStream(symbol?, options?);
// asks/bids: [price, quantity][] (number tuples)
```

### useMarkPrice / useMarkPriceBySymbol

```tsx
import { useMarkPrice, useMarkPriceBySymbol } from '@orderly.network/hooks';
const allMarkPrices = useMarkPrice(); // map of symbol -> price
const markPrice = useMarkPriceBySymbol(symbol); // number | undefined for one symbol
```

### useTickerStream

24h ticker stats for all symbols.

```tsx
import { useTickerStream } from '@orderly.network/hooks';
const tickers = useTickerStream(); // Record<symbol, Ticker> | undefined
// Ticker: { symbol, last_price, high_24h, low_24h, volume_24h,
//   quote_volume_24h, open, price_change_percent_24h, ... }
```

### useSymbolInfo / useSymbolsInfo

Trading rules for a symbol.

```tsx
import { useSymbolInfo, useSymbolsInfo } from '@orderly.network/hooks';
const symbolInfo = useSymbolInfo(symbol); // API.SymbolExt
// { symbol, base, quote, base_min, base_max, quote_tick, price_range, leverage_max, ... }
```

### useMarkets / useMarketList

Market metadata listings.

```tsx
import { useMarkets, useMarketList } from '@orderly.network/hooks';
const markets = useMarkets(); // API.MarketInfo[] | undefined
```

---

## Balance & Collateral Hooks

### useCollateral

Account collateral and portfolio value. Takes `{ dp }` (decimal places).

```tsx
import { useCollateral } from '@orderly.network/hooks';

const {
  totalCollateral, // total account collateral value
  freeCollateral, // available for new positions
  freeCollateralUSDCOnly, // free collateral in USDC only
  totalValue, // number | null — total portfolio value
  availableBalance, // withdrawable
  unsettledPnL, // unrealized PnL across positions
  holding, // API.Holding[]
  accountInfo, // API.AccountInfo
  usdcHolding,
} = useCollateral({ dp: 4 });

// Example
function AccountSummary() {
  const { totalCollateral, freeCollateral } = useCollateral({ dp: 2 });
  return (
    <p>
      Total: {totalCollateral} · Free: {freeCollateral}
    </p>
  );
}
```

> There is **no `useBalance` hook**. For per-token holdings use `useCollateral().holding`, or `useHoldingStream` for the real-time holding feed.

### useHoldingStream

Real-time holdings feed.

```tsx
import { useHoldingStream } from '@orderly.network/hooks';
const holdings = useHoldingStream(); // API.Holding[] | undefined
```

---

## Chain Hooks

### useChains

Supported chains. Returns `[chains, helpers]`.

```tsx
import { useChains } from '@orderly.network/hooks';
const [chains, { findByChainId }] = useChains();
// chains: API.Chain[] (id, name, network, chain_id, explorer, ...)
```

### useMarginRatio

```tsx
import { useMarginRatio } from '@orderly.network/hooks';
const { currentLeverage, marginRatio, mmr, maintenanceMargin } = useMarginRatio();
```

---

## Deposit / Withdraw Hooks

### useDeposit

Handle deposits (allowance + deposit). `deposit()` takes no argument — it uses the `quantity` set via `setQuantity`.

```tsx
import { useDeposit } from '@orderly.network/hooks';

const {
  balance,              // string | null — source wallet balance
  allowance,            // string — ERC-20 allowance
  depositFee,           // bigint — in wei
  isNativeToken,
  dst,                  // { symbol, address, decimals, chainId, network }
  targetChain,          // API.Chain
  quantity, setQuantity,
  approve,              // (amount?) => Promise<void>
  deposit,              // () => Promise<any>
  fetchBalance,         // (address, decimals?) => Promise<string>
  fetchBalances,        // (tokens) => Promise<Record<string,string>>
  balanceRevalidating, allowanceRevalidating, depositFeeRevalidating,
} = useDeposit({
  // all optional
  srcChainId?, srcToken?, dstToken?, address?, decimals?,
  crossChainRouteAddress?, depositorAddress?,
});

// Example
function DepositUSDC() {
  const { balance, allowance, approve, deposit, setQuantity } = useDeposit({
    srcChainId: 42161, srcToken: 'USDC',
    address: '0xaf88d065e77c8cC2239327C5EDb3A432268e5831', decimals: 6,
  });
  const handleDeposit = async () => {
    setQuantity('100');
    if (Number(allowance) < 100) await approve();
    await deposit();
  };
  return <button onClick={handleDeposit}>Deposit</button>;
}
```

### useWithdraw

Handle withdrawals. `withdraw()` takes the full inputs object.

```tsx
import { useWithdraw } from '@orderly.network/hooks';

const {
  dst,                  // { symbol, address, decimals, chainId, network }
  withdraw,             // (inputs) => Promise<any>
  maxAmount,            // number — max withdrawable (use this)
  unsettledPnL, availableBalance,
} = useWithdraw({ srcChainId?, token?, decimals? });

// Example
await withdraw({
  chainId: 42161,
  token: 'USDC',
  amount: '100',
  allowCrossChainWithdraw: false,
  receiver: '0x...',   // optional
});
```

> `availableWithdraw` is deprecated — use `maxAmount`.

---

## Leverage Hooks

Account-level leverage (`useLeverage`) is **not** symbol-scoped. Use `useSymbolLeverage` / `useMaxLeverage` for a specific symbol.

```tsx
import { useLeverage, useSymbolLeverage, useMaxLeverage } from '@orderly.network/hooks';

// Account-level
const {
  update, // ({ leverage }) => Promise<{ max_leverage } | undefined>
  curLeverage,
  maxLeverage,
  leverageLevers,
  isLoading,
} = useLeverage();

// Symbol-level
const { maxLeverage, update, isLoading } = useSymbolLeverage(symbol);

const maxLev = useMaxLeverage(symbol); // number | "-"

// Example
function LeverageControl({ symbol }: { symbol: string }) {
  const { maxLeverage, update, isLoading } = useSymbolLeverage(symbol);
  return (
    <input
      type="range"
      min={1}
      max={maxLeverage}
      onChange={(e) => update({ leverage: parseInt(e.target.value), symbol })}
    />
  );
}
```

---

## Common Patterns

### Trading Interface

```tsx
import {
  useAccount,
  useOrderEntry,
  usePositionStream,
  useOrderbookStream,
  useCollateral,
} from '@orderly.network/hooks';
import { AccountStatusEnum, OrderSide, OrderType } from '@orderly.network/types';

function TradingInterface({ symbol }: { symbol: string }) {
  const { state } = useAccount();
  const [positions] = usePositionStream();
  const { asks, bids } = useOrderbookStream(symbol);
  const { freeCollateral } = useCollateral({ dp: 2 });
  const { submit, setValue, helper, isMutating, formattedOrder } = useOrderEntry(symbol);

  if (state.status < AccountStatusEnum.Connected) return <ConnectWallet />;

  // ...render orderbook, order form, positions...
}
```

> **Tip**: For a full trading UI without writing this by hand, use `<TradingPage>` from `@orderly.network/trading` (see `orderly-sdk-page-components`). The hooks above are for custom UI.

## Related Skills

- **orderly-sdk-ui-components** - Pre-built UI components
- **orderly-sdk-trading-workflows** - End-to-end trading flows
- **orderly-sdk-wallet-connection** - Wallet + account state machine
- **orderly-trading-orders** - Order management details (REST)
- **orderly-positions-tpsl** - Position management (REST)
- **orderly-websocket-streaming** - Underlying WebSocket implementation
