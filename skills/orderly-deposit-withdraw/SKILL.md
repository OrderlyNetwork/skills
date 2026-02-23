---
name: orderly-deposit-withdraw
description: Handle token deposits and withdrawals across chains, including allowance approval, vault interactions, and cross-chain operations
---

# Orderly Network: Deposit & Withdraw

This skill covers depositing and withdrawing assets on Orderly Network, including token approvals, vault interactions, and cross-chain operations.

## When to Use

- Building deposit/withdrawal interfaces
- Managing collateral
- Implementing cross-chain transfers
- Handling token approvals

## Prerequisites

- Connected Web3 wallet
- Tokens on supported chain
- Understanding of ERC20 approve pattern

## Supported Chains & Tokens

| Chain    | Chain ID  | USDC | USDT |
| -------- | --------- | ---- | ---- |
| Arbitrum | 42161     | ✅   | ✅   |
| Optimism | 10        | ✅   | ✅   |
| Base     | 8453      | ✅   | ✅   |
| Ethereum | 1         | ✅   | ✅   |
| Mantle   | 5000      | ✅   | -    |
| Solana   | 900900900 | ✅   | -    |

## Deposit Flow

```
1. Check wallet balance
2. Approve token allowance (if needed)
3. Execute deposit transaction
4. Wait for confirmation
5. Balance updates on Orderly
```

## React SDK: useDeposit Hook

```typescript
import { useDeposit, useWalletConnector } from '@orderly.network/hooks';

function DepositForm() {
  const { connectedChain } = useWalletConnector();

  const {
    balance,          // Wallet balance
    allowance,        // Current token allowance
    approve,          // Approve function
    deposit,          // Deposit function
    depositFee,       // Estimated fee
    setQuantity,      // Set deposit amount
    fetchBalance,     // Refresh balance
    isApproving,
    isDepositing,
  } = useDeposit({
    address: '0xaf88d065e77c8cC2239327C5EDb3A432268e5831', // USDC on Arbitrum
    decimals: 6,
    srcToken: 'USDC',
    srcChainId: Number(connectedChain?.id) || 42161,
  });

  const [amount, setAmount] = useState('');

  const handleApprove = async () => {
    await approve(); // Approves MaxUint256 by default
  };

  const handleDeposit = async () => {
    setQuantity(amount);
    const result = await deposit();
    console.log('Deposit successful:', result);
  };

  return (
    <div className="deposit-form">
      <div className="balance">
        Wallet Balance: {balance} USDC
        <button onClick={() => fetchBalance()}>Refresh</button>
      </div>

      <input
        type="text"
        placeholder="Amount"
        value={amount}
        onChange={(e) => {
          setAmount(e.target.value);
          setQuantity(e.target.value);
        }}
      />

      <div className="fee">
        Deposit Fee: {depositFee} USDC
      </div>

      {parseFloat(allowance) < parseFloat(amount) ? (
        <button onClick={handleApprove} disabled={isApproving}>
          {isApproving ? 'Approving...' : 'Approve USDC'}
        </button>
      ) : (
        <button onClick={handleDeposit} disabled={isDepositing}>
          {isDepositing ? 'Depositing...' : 'Deposit'}
        </button>
      )}
    </div>
  );
}
```

## REST API: Get Deposit Info

```typescript
// Get supported tokens with collateral factors
GET /v1/public/token

// Response
{
  "success": true,
  "data": [
    {
      "token": "USDC",
      "decimals": 6,
      "address": {
        "42161": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
        "10": "0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85"
      },
      "collateral_factor": 1.0  // USDC counts as 100% collateral
    },
    {
      "token": "USDT",
      "decimals": 6,
      "address": { ... },
      "collateral_factor": 0.95  // USDT counts as 95% collateral
    }
  ]
}

// Get chain info
GET /v1/public/chain_info

// Get vault balance (TVL)
GET /v1/public/vault_balance
```

### Understanding Collateral Factors

Each token has a **collateral factor** (0.0 to 1.0) that determines how much of your deposit counts toward trading collateral:

| Token | Collateral Factor | $1000 Deposit = Collateral Value |
| ----- | ----------------- | -------------------------------- |
| USDC  | 1.0 (100%)        | $1000                            |
| USDT  | 0.95 (95%)        | $950                             |
| Other | Varies            | Depends on risk assessment       |

**Example**: If you deposit $1000 USDT with a 0.95 collateral factor, only $950 counts as collateral for margin calculations.

**Why it matters**: Lower collateral factors require larger deposits to maintain the same position size. Always check the current collateral factor before depositing alternative tokens.

## Direct Contract Interaction

### Get Vault Address

```typescript
import { getContractAddresses } from '@orderly.network/contracts';

const addresses = getContractAddresses('arbitrum', 'mainnet');
// {
//   vault: '0x816f722424B49Cf1275cc86DA9840Fbd5a6167e9',
//   usdc: '0xaf88d065e77c8cC2239327C5EDb3A432268e5831',
//   usdt: '0xFd086bC7CD5C481DCC9C85ebE478A1C0b69FCbb9'
// }
```

### Deposit via Contract

```typescript
import { ethers } from 'ethers';

// Token ABI (minimal for approve)
const ERC20_ABI = [
  'function approve(address spender, uint256 amount) returns (bool)',
  'function allowance(address owner, address spender) view returns (uint256)',
  'function balanceOf(address owner) view returns (uint256)',
];

// Vault ABI
const VAULT_ABI = ['function deposit(address token, uint256 amount) external payable'];

async function depositUSDC(
  provider: ethers.BrowserProvider,
  amount: string,
  vaultAddress: string,
  usdcAddress: string
) {
  const signer = await provider.getSigner();
  const userAddress = await signer.getAddress();

  // Token contract
  const usdc = new ethers.Contract(usdcAddress, ERC20_ABI, signer);

  // Check and approve
  const amountWei = ethers.parseUnits(amount, 6);
  const allowance = await usdc.allowance(userAddress, vaultAddress);

  if (allowance < amountWei) {
    console.log('Approving USDC...');
    const approveTx = await usdc.approve(vaultAddress, ethers.MaxUint256);
    await approveTx.wait();
    console.log('Approved');
  }

  // Deposit
  const vault = new ethers.Contract(vaultAddress, VAULT_ABI, signer);
  console.log('Depositing...');
  const depositTx = await vault.deposit(usdcAddress, amountWei);
  const receipt = await depositTx.wait();
  console.log('Deposited:', receipt.hash);

  return receipt;
}
```

## Withdrawal Flow

```
1. Request withdrawal nonce
2. Sign withdrawal request
3. Submit signed request
4. Wait for processing
5. Receive tokens on chain
```

## React SDK: useWithdraw Hook

```typescript
import { useWithdraw } from '@orderly.network/hooks';

function WithdrawForm() {
  const {
    withdraw,
    isLoading,
    withdrawFee,
    estimatedTime,
  } = useWithdraw();

  const [amount, setAmount] = useState('');
  const [destination, setDestination] = useState('');

  const handleWithdraw = async () => {
    try {
      const result = await withdraw({
        symbol: 'USDC',
        amount: amount,
        address: destination, // Destination address
        chainId: 42161, // Arbitrum
        network: 'arbitrum',
      });
      console.log('Withdrawal initiated:', result);
    } catch (error) {
      console.error('Withdrawal failed:', error);
    }
  };

  return (
    <div className="withdraw-form">
      <input
        type="text"
        placeholder="Amount"
        value={amount}
        onChange={(e) => setAmount(e.target.value)}
      />

      <input
        type="text"
        placeholder="Destination Address"
        value={destination}
        onChange={(e) => setDestination(e.target.value)}
      />

      <div className="info">
        Fee: {withdrawFee} USDC
        <br />
        Estimated Time: {estimatedTime}
      </div>

      <button onClick={handleWithdraw} disabled={isLoading}>
        {isLoading ? 'Processing...' : 'Withdraw'}
      </button>
    </div>
  );
}
```

## REST API: Withdrawal

```typescript
// Step 1: Get withdrawal nonce
GET /v1/withdraw_nonce

// Response
{
  "success": true,
  "data": {
    "nonce": 123456,
    "expiry": 1699123456
  }
}

// Step 2: Create EIP-712 signature
const WITHDRAW_TYPES = {
  Withdraw: [
    { name: 'brokerId', type: 'string' },
    { name: 'chainId', type: 'uint256' },
    { name: 'receiver', type: 'address' },
    { name: 'token', type: 'address' },
    { name: 'amount', type: 'uint256' },
    { name: 'withdrawNonce', type: 'uint256' },
    { name: 'timestamp', type: 'uint64' },
  ],
};

const withdrawMessage = {
  brokerId: 'woofi_dex',
  chainId: 42161,
  receiver: '0x...', // Destination address
  token: '0xaf88d065e77c8cC2239327C5EDb3A432268e5831', // USDC
  amount: '10000000', // 10 USDC (6 decimals)
  withdrawNonce: 123456,
  timestamp: Date.now(),
};

// Step 3: Sign and submit
const signature = await wallet.signTypedData(domain, WITHDRAW_TYPES, withdrawMessage);

POST /v1/withdraw_request
Body: {
  message: withdrawMessage,
  signature: signature,
  userAddress: '0x...',
}

// Note: For cross-chain withdrawals, set allow_cross_chain_withdrawal: true
```

## Cross-Chain Withdrawal

```typescript
// Withdraw to different chain than deposit
POST /v1/withdraw_request
Body: {
  message: {
    ...withdrawMessage,
    chainId: 10, // Withdraw to Optimism
  },
  signature: signature,
  userAddress: '0x...',
  allow_cross_chain_withdrawal: true,
}

// Response includes cross-chain details
{
  "success": true,
  "data": {
    "withdrawal_id": "wd_123",
    "status": "PROCESSING",
    "estimated_time": "30 minutes",
    "source_chain": 42161,
    "destination_chain": 10
  }
}
```

## Internal Transfer

Transfer between accounts on Orderly (instant, no gas):

```typescript
// Get transfer nonce
GET /v1/transfer_nonce

// Internal transfer
POST /v1/internal_transfer
Body: {
  from_account_id: 'your_account_id',
  to_account_id: 'recipient_account_id',
  token: 'USDC',
  amount: '100.00',
}

// V2 endpoint with more options
POST /v2/internal_transfer
Body: {
  from_account_id: 'your_account_id',
  to_account_id: 'recipient_account_id',
  token: 'USDC',
  amount: '100.00',
  memo: 'Optional memo',
}
```

## Asset History

```typescript
// Get deposit/withdrawal history
GET /v1/asset/history
Query params:
  - token: USDC (optional)
  - side: DEPOSIT | WITHDRAW (optional)
  - start: timestamp
  - end: timestamp
  - page: 1
  - size: 20

// Response
{
  "success": true,
  "data": {
    "rows": [
      {
        "id": "tx_123",
        "token": "USDC",
        "side": "DEPOSIT",
        "amount": "1000.00",
        "status": "COMPLETED",
        "tx_hash": "0x...",
        "chain_id": 42161,
        "created_at": 1699123456
      }
    ],
    "total": 50
  }
}
```

## Testnet Faucet

```typescript
// Get testnet USDC (testnet only)
POST /v1/faucet/usdc

// EVM Testnet
POST https://testnet-operator-evm.orderly.org/v1/faucet/usdc

// Solana Testnet
POST https://testnet-operator-sol.orderly.org/v1/faucet/usdc

// Each account can use faucet max 5 times
// EVM: 1000 USDC per request
// Solana: 100 USDC per request
```

## Common Issues

### "Insufficient allowance" error

- Call `approve()` before deposit
- Check if approval transaction confirmed

### "Insufficient balance" error

- Check wallet balance, not just Orderly balance
- Ensure correct token decimals

### Withdrawal stuck in "PROCESSING"

- Cross-chain withdrawals take 30+ minutes
- Check `/v1/asset/history` for status
- Contact support if stuck > 24 hours

### "Cross-chain withdrawal required" error

- Set `allow_cross_chain_withdrawal: true`
- Destination chain differs from current chain

## Related Skills

- **orderly-getting-started** - Account setup
- **orderly-contract-addresses** - Vault/token addresses
- **orderly-api-authentication** - Signing withdrawal requests
