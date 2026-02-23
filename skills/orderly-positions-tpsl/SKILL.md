---
name: orderly-positions-tpsl
description: Monitor positions in real-time, configure Take-Profit/Stop-Loss orders, and manage risk with leverage settings
---

# Orderly Network: Positions & TP/SL

This skill covers position management, PnL tracking, leverage settings, and configuring Take-Profit (TP) and Stop-Loss (SL) orders for risk management.

## When to Use

- Building a position management interface
- Implementing risk management with TP/SL
- Tracking unrealized PnL
- Adjusting leverage settings

## Prerequisites

- Open positions on Orderly
- Understanding of perpetual futures
- API key with `read` and `trading` scopes

## Position Data Structure

```typescript
interface Position {
  symbol: string; // e.g., "PERP_ETH_USDC"
  position_qty: number; // Positive = long, Negative = short
  average_open_price: number; // Entry price
  mark_price: number; // Current mark price
  unrealized_pnl: number; // Unrealized profit/loss
  unrealized_pnl_roi: number; // ROI percentage
  mmr: number; // Maintenance margin ratio
  imr: number; // Initial margin ratio
  notional: number; // Position value
  leverage: number; // Current leverage
  liq_price: number; // Liquidation price
  bank_cost: number; // Position cost
  settle_i_owe: number; // Unsettled debt
}
```

## Get Positions (REST API)

```typescript
// Get all positions
GET /v1/positions

// Get position for specific symbol
GET /v1/position/{symbol}

// Example response
{
  "success": true,
  "data": {
    "rows": [
      {
        "symbol": "PERP_ETH_USDC",
        "position_qty": 0.5,
        "average_open_price": 3000,
        "mark_price": 3100,
        "unrealized_pnl": 50,
        "unrealized_pnl_roi": 0.0333,
        "mmr": 0.01,
        "imr": 0.02,
        "notional": 1550,
        "leverage": 10,
        "liq_price": 2700
      }
    ]
  }
}
```

## React SDK: usePositionStream

Stream positions in real-time with automatic PnL updates:

```typescript
import { usePositionStream } from '@orderly.network/hooks';

function PositionsTable() {
  const {
    rows,
    aggregated,
    totalUnrealizedROI,
    isLoading
  } = usePositionStream();

  if (isLoading) return <div>Loading positions...</div>;

  return (
    <div>
      <div className="summary">
        <h3>Total Unrealized PnL: {aggregated?.totalUnrealizedPnl?.toFixed(2)} USDC</h3>
        <p>ROI: {(totalUnrealizedROI * 100).toFixed(2)}%</p>
      </div>

      <table>
        <thead>
          <tr>
            <th>Symbol</th>
            <th>Size</th>
            <th>Entry Price</th>
            <th>Mark Price</th>
            <th>Unrealized PnL</th>
            <th>Leverage</th>
            <th>Liq. Price</th>
          </tr>
        </thead>
        <tbody>
          {rows.map((position) => (
            <tr key={position.symbol}>
              <td>{position.symbol}</td>
              <td className={position.position_qty > 0 ? 'long' : 'short'}>
                {position.position_qty > 0 ? '+' : ''}{position.position_qty}
              </td>
              <td>{position.average_open_price.toFixed(2)}</td>
              <td>{position.mark_price.toFixed(2)}</td>
              <td className={position.unrealized_pnl >= 0 ? 'profit' : 'loss'}>
                {position.unrealized_pnl.toFixed(2)} USDC
              </td>
              <td>{position.leverage}x</td>
              <td>{position.liq_price.toFixed(2)}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

## Close Position

### Partial Close

```typescript
import { usePositionClose } from '@orderly.network/hooks';

function ClosePositionButton({ symbol, positionQty }: { symbol: string; positionQty: number }) {
  const { closePosition, isClosing } = usePositionClose();

  const handleClose = async (percentage: number) => {
    const quantity = Math.abs(positionQty) * (percentage / 100);
    await closePosition({
      symbol,
      qty: quantity,
      side: positionQty > 0 ? 'SELL' : 'BUY',
    });
  };

  return (
    <div>
      <button onClick={() => handleClose(25)} disabled={isClosing}>Close 25%</button>
      <button onClick={() => handleClose(50)} disabled={isClosing}>Close 50%</button>
      <button onClick={() => handleClose(100)} disabled={isClosing}>Close 100%</button>
    </div>
  );
}
```

### Market Close (REST API)

```typescript
// Close entire position at market price
POST /v1/order
Body: {
  symbol: 'PERP_ETH_USDC',
  side: positionQty > 0 ? 'SELL' : 'BUY',
  order_type: 'MARKET',
  order_quantity: Math.abs(positionQty).toString(),
  reduce_only: true,
}
```

## Leverage Management

### Get Current Leverage

```typescript
GET /v1/client/leverage?symbol={symbol}

// Response
{
  "success": true,
  "data": {
    "leverage": 10,
    "max_leverage": 25
  }
}
```

### Set Leverage

```typescript
POST /v1/client/leverage
Body: {
  symbol: 'PERP_ETH_USDC',
  leverage: 15,  // New leverage value
}

// React SDK
import { useLeverage } from '@orderly.network/hooks';

function LeverageSlider({ symbol }: { symbol: string }) {
  const { leverage, maxLeverage, setLeverage, isLoading } = useLeverage(symbol);

  const handleChange = async (newLeverage: number) => {
    try {
      await setLeverage(newLeverage);
    } catch (error) {
      console.error('Failed to set leverage:', error);
    }
  };

  return (
    <div>
      <label>Leverage: {leverage}x</label>
      <input
        type="range"
        min="1"
        max={maxLeverage}
        value={leverage}
        onChange={(e) => handleChange(parseInt(e.target.value))}
        disabled={isLoading}
      />
    </div>
  );
}
```

## Take-Profit / Stop-Loss Orders

### TP/SL Order Types

| Type            | Description                                |
| --------------- | ------------------------------------------ |
| `TAKE_PROFIT`   | Trigger when price reaches target (profit) |
| `STOP_LOSS`     | Trigger when price drops below threshold   |
| `TRAILING_STOP` | Dynamic stop that follows price            |

### Using useTPSLOrder Hook

```typescript
import { useTPSLOrder } from '@orderly.network/hooks';

function TPSSettings({ position }: { position: Position }) {
  const [computed, { setValue, submit, validate, reset }] = useTPSLOrder(position);

  const handleSubmit = async () => {
    try {
      await validate();
      await submit();
      console.log('TP/SL order placed');
    } catch (error) {
      console.error('TP/SL failed:', error);
    }
  };

  return (
    <div className="tpsl-form">
      <h4>Take Profit</h4>
      <div>
        <label>Trigger Price</label>
        <input
          type="number"
          placeholder="TP Price"
          onChange={(e) => setValue('tp_trigger_price', e.target.value)}
        />
      </div>
      <div>
        <label>Or Offset %</label>
        <input
          type="number"
          placeholder="e.g., 5 for 5%"
          onChange={(e) => setValue('tp_offset_percentage', parseFloat(e.target.value))}
        />
      </div>

      <h4>Stop Loss</h4>
      <div>
        <label>Trigger Price</label>
        <input
          type="number"
          placeholder="SL Price"
          onChange={(e) => setValue('sl_trigger_price', e.target.value)}
        />
      </div>
      <div>
        <label>Or Offset %</label>
        <input
          type="number"
          placeholder="e.g., -5 for -5%"
          onChange={(e) => setValue('sl_offset_percentage', parseFloat(e.target.value))}
        />
      </div>

      <button onClick={handleSubmit}>Set TP/SL</button>
    </div>
  );
}
```

### REST API: Algo Orders for TP/SL

```typescript
// Place TP/SL for existing position
POST /v1/algo/order
Body: {
  symbol: 'PERP_ETH_USDC',
  type: 'CLOSE_POSITION',
  algoType: 'TAKE_PROFIT',  // or 'STOP_LOSS'
  trigger_price: '3500',
  quantity: '0.5',  // Position size to close
  price: '3500',    // Limit price (optional, default = trigger_price)
}

// Positional TP/SL (attached to position)
POST /v1/algo/order
Body: {
  symbol: 'PERP_ETH_USDC',
  type: 'CLOSE_POSITION',
  childOrder: [
    {
      type: 'CLOSE_POSITION',
      algoType: 'TAKE_PROFIT',
      trigger_price: '3500',
      quantity: '0.5',
    },
    {
      type: 'CLOSE_POSITION',
      algoType: 'STOP_LOSS',
      trigger_price: '2800',
      quantity: '0.5',
    },
  ],
}
```

### Trailing Stop

```typescript
POST /v1/algo/order
Body: {
  symbol: 'PERP_ETH_USDC',
  type: 'MARKET',
  algoType: 'TRAILING_STOP',
  trigger_price: '3000',    // Initial trigger
  callbackRate: '5',        // 5% trailing distance
  quantity: '0.5',
}

// The stop price will trail the mark price:
// If price goes to 3200, stop moves to 3040 (3200 * 0.95)
// If price goes to 3500, stop moves to 3325 (3500 * 0.95)
// If price drops to stop, position closes
```

### Cancel TP/SL Orders

```typescript
// Cancel single algo order
DELETE /v1/algo/order?order_id={order_id}&symbol={symbol}

// Cancel all algo orders for symbol
DELETE /v1/algo/orders?symbol={symbol}

// React SDK
const [algoOrders, { cancelAlgoOrder }] = useAlgoOrderStream();
await cancelAlgoOrder(orderId);
```

## Position History

```typescript
GET /v1/position_history?symbol={symbol}&start={timestamp}&end={timestamp}

// Response includes closed positions with realized PnL
```

## PnL Calculations

### Unrealized PnL

```typescript
// For LONG positions
unrealizedPnL = (markPrice - averageOpenPrice) * positionQty;

// For SHORT positions
unrealizedPnL = (averageOpenPrice - markPrice) * Math.abs(positionQty);
```

### ROI

```typescript
// Return on Investment
roi = unrealizedPnL / ((averageOpenPrice * Math.abs(positionQty)) / leverage);
```

### Liquidation Price

```typescript
// For LONG positions
liqPrice = averageOpenPrice * (1 - mmr - 1 / leverage);

// For SHORT positions
liqPrice = averageOpenPrice * (1 + mmr + 1 / leverage);
```

## Risk Metrics

```typescript
interface RiskMetrics {
  total_collateral: number; // Total account value
  free_collateral: number; // Available for new positions
  used_collateral: number; // Locked in positions
  margin_ratio: number; // Health indicator (higher = safer)
  total_unrealized_pnl: number; // Sum of all position PnL
  total_notional: number; // Total position value
}

// Fetch via
GET / v1 / client / holding;
```

## Common Issues

### "Position would exceed max leverage" error

- Current notional with new leverage exceeds limits
- Reduce position size or increase collateral

### "Insufficient margin for TP/SL" error

- TP/SL orders require available margin
- Close some positions or add collateral

### TP/SL not triggering

- Check if trigger_price is valid
- Verify the order is in NEW status
- Market may have gapped past trigger

### Liquidation risk

- Monitor margin_ratio closely
- Set stop-losses early
- Watch funding rates for extended positions

## Related Skills

- **orderly-trading-orders** - Placing orders
- **orderly-websocket-streaming** - Real-time position updates
- **orderly-sdk-react-hooks** - Full SDK reference
- **orderly-api-authentication** - Signing requests
