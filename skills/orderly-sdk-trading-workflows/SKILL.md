---
name: orderly-sdk-trading-workflows
description: Complete trading workflows for Orderly Network DEX applications, from wallet connection through order execution, position management, and withdrawal.
---

# Orderly Network: SDK Trading Workflows

End-to-end trading workflows: connect → deposit → trade → monitor → close → withdraw.

## When to Use

- Building a complete trading interface
- Implementing the deposit → trade → withdraw flow
- Building **custom** order/position UI on top of SDK hooks

## Prerequisites

- Orderly SDK packages installed (see `orderly-sdk-install-dependency`)
- Wallet connection configured (see `orderly-sdk-wallet-connection`)
- `OrderlyAppProvider` wrapping the app

## Overview

```
Connect → Deposit → Trade → Monitor → Close → Withdraw
```

> **Recommended path**: the reference DEX does not hand-roll the trading flow. It renders `<TradingPage>` (from `@orderly.network/trading`), which already contains the chart, orderbook, order entry, positions, and order management. The hook-based code below is for **custom** trading UIs where you replace parts of `TradingPage`.

## 1. Connect & Authenticate

`AccountStatusEnum` is exported from `@orderly.network/types` (not `hooks`). The full state machine is documented in `orderly-sdk-wallet-connection`.

```tsx
import { useAccount } from '@orderly.network/hooks';
import { AccountStatusEnum } from '@orderly.network/types';

function TradingGuard({ children }: { children: React.ReactNode }) {
  const { state, createAccount, createOrderlyKey } = useAccount();

  switch (state.status) {
    case AccountStatusEnum.NotConnected:
      return <ConnectWalletPrompt />;
    case AccountStatusEnum.Connected:
      return <button onClick={() => createAccount()}>Create Orderly Account</button>;
    case AccountStatusEnum.NotSignedIn:
      return <button onClick={() => createOrderlyKey()}>Enable Trading</button>;
    default:
      return <>{children}</>;
  }
}
```

### AuthGuard

For custom UI, gate content behind `AuthGuard` (from `@orderly.network/ui-connector`). Note: `TradingPage` manages its own auth prompts, so you do **not** wrap it in `AuthGuard`.

```tsx
import { AuthGuard } from '@orderly.network/ui-connector';

<AuthGuard>{/* protected custom UI */}</AuthGuard>;
```

## 2. Deposit

### `useDeposit`

`deposit()` takes **no argument** — it submits the amount set via `setQuantity`. Approve ERC-20 allowance first when the token is not native.

```tsx
import { useDeposit } from '@orderly.network/hooks';
import { toast } from '@orderly.network/ui';

function DepositForm() {
  const { balance, allowance, approve, deposit, quantity, setQuantity, depositFee, isNativeToken } =
    useDeposit({
      srcChainId: 42161,
      srcToken: 'USDC',
      address: '0xaf88d065e77c8cC2239327C5EDb3A432268e5831',
      decimals: 6,
    });

  const handleDeposit = async () => {
    setQuantity('100');
    try {
      // Approve if allowance is insufficient (allowance is a wei string)
      if (!isNativeToken) await approve();
      await deposit();
      toast.success('Deposit submitted');
    } catch (e) {
      toast.error((e as Error).message);
    }
  };

  return (
    <div>
      <input value={quantity} onChange={(e) => setQuantity(e.target.value)} placeholder="Amount" />
      <p>Wallet balance: {balance}</p>
      <p>Deposit fee (wei): {depositFee?.toString()}</p>
      <button onClick={handleDeposit}>Deposit</button>
    </div>
  );
}
```

> `depositFee` is a `bigint` (in wei). Use a pre-built form (`DepositForm`/`DepositFormWidget` from `@orderly.network/ui-transfer`) for a fully styled experience.

## 3. Check Account Balance

```tsx
import { useCollateral } from '@orderly.network/hooks';

function AccountSummary() {
  const { totalValue, freeCollateral, availableBalance, unsettledPnL } = useCollateral({ dp: 2 });
  // totalValue: number | null — total portfolio value
  // freeCollateral: available for new positions
  // availableBalance: withdrawable
  // unsettledPnL: unrealized PnL across positions
}
```

## 4. Place Orders

### Recommended: `TradingPage`

```tsx
import { TradingPage } from '@orderly.network/trading';
<TradingPage symbol="PERP_BTC_USDC" tradingViewConfig={config.tradingPage.tradingViewConfig} />;
```

### Custom UI: `useOrderEntry`

Set order fields with `setValue`/`setValues` (keys: `side`, `order_type`, `order_price`, `order_quantity`, `reduce_only`, `tp_trigger_price`, `sl_price`), validate, then `submit()`. Validation state lives in `metaState` (not top-level `errors`).

```tsx
import { useOrderEntry } from '@orderly.network/hooks';
import { OrderSide, OrderType } from '@orderly.network/types';

function OrderForm({ symbol }: { symbol: string }) {
  const { submit, setValues, setValue, helper, isMutating, metaState, formattedOrder } =
    useOrderEntry(symbol);

  useEffect(() => {
    setValues({ side: OrderSide.BUY, order_type: OrderType.LIMIT });
  }, [symbol]);

  const handleSubmit = async () => {
    const result = await helper.validate();
    if (result) return; // validation issues are in metaState.errors
    try {
      await submit({ resetOnSuccess: true });
      toast.success('Order submitted');
    } catch (e) {
      toast.error((e as Error).message);
    }
  };

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        handleSubmit();
      }}
    >
      <input
        placeholder="Price"
        value={formattedOrder.order_price ?? ''}
        onChange={(e) => setValue('order_price', e.target.value)}
      />
      <input
        placeholder="Quantity"
        value={formattedOrder.order_quantity ?? ''}
        onChange={(e) => setValue('order_quantity', e.target.value)}
      />
      <button type="submit" disabled={isMutating}>
        {isMutating ? 'Placing…' : 'Buy'}
      </button>
    </form>
  );
}
```

### Low-level: `useMutation`

For non-orderly-entry flows (e.g. a bot), post directly. This is **not** what the DEX template uses.

```tsx
import { useMutation } from '@orderly.network/hooks';
import { OrderType, OrderSide } from '@orderly.network/types';

const [submitOrder] = useMutation('/v1/order');
await submitOrder({
  symbol,
  side,
  order_type: OrderType.MARKET,
  order_quantity: quantity,
});
```

## 5. Monitor & Cancel Orders

`useOrderStream` returns `[orders, actions]`. Use the `OrderStatus` enum — **not** the string `'OPEN'` (open orders are `OrderStatus.INCOMPLETE`).

```tsx
import { useOrderStream } from '@orderly.network/hooks';
import { OrderStatus } from '@orderly.network/types';

function OpenOrders({ symbol }: { symbol?: string }) {
  const [orders, { cancelOrder, cancelAllPendingOrders }] = useOrderStream({
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

## 6. Monitor Positions

`usePositionStream` returns a **3-tuple** `[positions, calc, status]`. The third element exposes `isLoading` (not `loading`).

```tsx
import { usePositionStream } from '@orderly.network/hooks';

function PositionsTable() {
  const [positions, , { isLoading }] = usePositionStream();
  return (
    <table>
      {(positions?.rows ?? []).map((p) => (
        <tr key={p.symbol}>
          <td>{p.symbol}</td>
          <td>{p.position_qty}</td>
          <td>{p.average_open_price}</td>
          <td>{p.mark_price}</td>
          <td className={p.unrealized_pnl >= 0 ? 'profit' : 'loss'}>
            {p.unrealized_pnl?.toFixed(2)}
          </td>
        </tr>
      ))}
    </table>
  );
}
```

## 7. Close a Position

Submit a reducing market order via `useOrderEntry` (set `reduce_only: true` and the opposite side):

```tsx
const { setValues, submit } = useOrderEntry(position.symbol);
await setValues({
  side: position.position_qty > 0 ? OrderSide.SELL : OrderSide.BUY,
  order_type: OrderType.MARKET,
  order_quantity: Math.abs(position.position_qty),
  reduce_only: true,
});
await submit();
```

Or use `usePositionClose` / the positions widget's close action for a one-call close.

## 8. Withdraw

`useWithdraw` returns `withdraw(inputs)` — pass the full inputs object (not a bare amount). Use `maxAmount` (not the deprecated `availableWithdraw`).

```tsx
import { useWithdraw } from '@orderly.network/hooks';

const { withdraw, maxAmount, unsettledPnL, availableBalance, dst } = useWithdraw({
  srcChainId: 42161,
  token: 'USDC',
  decimals: 6,
});

await withdraw({
  chainId: 42161,
  token: 'USDC',
  amount: '100',
  allowCrossChainWithdraw: false,
  receiver: '0x…', // optional
});
```

For a styled UI, use `WithdrawForm` / `WithdrawWidget` from `@orderly.network/ui-transfer`.

## 9. Leverage

Account-level leverage (`useLeverage`) is **not** symbol-scoped. For the active symbol use `useSymbolLeverage`.

```tsx
import { useLeverage, useSymbolLeverage } from '@orderly.network/hooks';

// Account-level
const { curLeverage, maxLeverage, update, isLoading } = useLeverage();
await update({ leverage: 5 });

// Symbol-level (preferred on the trading page)
const { maxLeverage, update: setSymbolLeverage } = useSymbolLeverage(symbol);
await setSymbolLeverage({ leverage: 5, symbol });
```

## 10. Risk Monitoring

```tsx
import { useMarginRatio } from '@orderly.network/hooks';
const { marginRatio, mmr, currentLeverage, maintenanceMargin } = useMarginRatio();
```

## Complete Trading Page (recommended)

The template's trading page is just `TradingPage` with config from `useOrderlyConfig()` — no manual auth guard, no manual order wiring:

```tsx
// app/pages/perp/Symbol.tsx
import { TradingPage } from '@orderly.network/trading';
import { useOrderlyConfig } from '@/utils/config';

export default function PerpSymbol() {
  const params = useParams();
  const [symbol, setSymbol] = useState(params.symbol!);
  const config = useOrderlyConfig();
  return (
    <TradingPage
      symbol={symbol}
      onSymbolChange={(data) => {
        setSymbol(data.symbol);
        navigate(`/perp/${data.symbol}`);
      }}
      tradingViewConfig={config.tradingPage.tradingViewConfig}
      sharePnLConfig={config.tradingPage.sharePnLConfig}
    />
  );
}
```

Route is `/perp/:symbol` (not `/trade`).

## Best Practices

1. **Prefer `TradingPage`** over hand-rolled order/position UI unless you need deep customization.
2. **Use the `OrderStatus` enum** (`INCOMPLETE`) for open-order streams — never the string `'OPEN'`.
3. **Read validation from `metaState`** (`metaState.errors`, `metaState.validated`) when using `useOrderEntry`.
4. **Approve before depositing** non-native ERC-20 tokens; check `isNativeToken`.
5. **Pass the full inputs object** to `withdraw()`, and use `maxAmount` for the withdrawable ceiling.

## Related Skills

- **orderly-sdk-wallet-connection** - Wallet + account state machine
- **orderly-sdk-react-hooks** - Full hook reference
- **orderly-sdk-page-components** - `TradingPage` and portfolio pages
- **orderly-trading-orders** - Order management (REST API)
- **orderly-positions-tpsl** - Position management (REST API)
- **orderly-deposit-withdraw** - Fund management (REST API)
