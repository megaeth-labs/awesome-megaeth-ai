# DeFi Trading Bot Skill for MegaETH

> AI coding skill for building automated DeFi trading bots on MegaETH using Kumbaya DEX (Uniswap V3 fork) with sub-10ms block times.

## Overview

MegaETH's 10ms block times enable trading strategies that are impossible on other chains. This skill teaches AI coding assistants how to build, deploy, and manage on-chain trading bots on MegaETH using real-time price feeds, instant transaction confirmations via `eth_sendRawTransactionSync` (EIP-7966), and Kumbaya DEX for execution.

## Concepts

### Why MegaETH for trading bots?
- **10ms block times** — strategies can react to price changes in real-time
- **Instant receipts** via `eth_sendRawTransactionSync` — no waiting for confirmations
- **Low gas** — high-frequency strategies are economically viable
- **Kumbaya DEX** — Uniswap V3 fork with concentrated liquidity

### Architecture
```
Price Feed (WebSocket) → Strategy Engine → Kumbaya DEX Router → MegaETH
     ↑                                                              |
     └────────── Position Monitor ←── Event Listener ←──────────────┘
```

## Key Patterns

### 1. Real-time price monitoring via WebSocket

```typescript
import { createPublicClient, webSocket } from 'viem';
import { megaeth } from './chains';

const client = createPublicClient({
  chain: megaeth,
  transport: webSocket('wss://rpc.megaeth.com/ws'),
});

// Subscribe to mini-blocks for real-time price updates
client.watchBlocks({
  onBlock: async (block) => {
    const price = await getPoolPrice(POOL_ADDRESS);
    await strategy.onPriceUpdate(price, block.timestamp);
  },
});
```

### 2. Instant swap execution

```typescript
import { encodeFunctionData } from 'viem';

const KUMBAYA_ROUTER = '0x...'; // Kumbaya SwapRouter address

async function executeSwap(tokenIn: string, tokenOut: string, amountIn: bigint, slippageBps: number) {
  const minAmountOut = await getQuote(tokenIn, tokenOut, amountIn);
  const minWithSlippage = minAmountOut * BigInt(10000 - slippageBps) / 10000n;

  const data = encodeFunctionData({
    abi: SWAP_ROUTER_ABI,
    functionName: 'exactInputSingle',
    args: [{
      tokenIn,
      tokenOut,
      fee: 3000, // 0.3% fee tier
      recipient: walletAddress,
      deadline: BigInt(Math.floor(Date.now() / 1000) + 60),
      amountIn,
      amountOutMinimum: minWithSlippage,
      sqrtPriceLimitX96: 0n,
    }],
  });

  // Use MegaETH instant confirmation
  const hash = await walletClient.sendTransaction({
    to: KUMBAYA_ROUTER,
    data,
  });

  // Receipt available instantly on MegaETH
  const receipt = await client.waitForTransactionReceipt({ hash });
  return receipt;
}
```

### 3. Simple momentum strategy

```typescript
interface PricePoint {
  price: number;
  timestamp: number;
}

class MomentumStrategy {
  private prices: PricePoint[] = [];
  private readonly windowSize = 20; // 20 blocks = 200ms on MegaETH
  private readonly threshold = 0.001; // 0.1% price movement

  async onPriceUpdate(price: number, timestamp: number) {
    this.prices.push({ price, timestamp });
    if (this.prices.length > this.windowSize) {
      this.prices.shift();
    }

    if (this.prices.length < this.windowSize) return;

    const oldest = this.prices[0].price;
    const momentum = (price - oldest) / oldest;

    if (momentum > this.threshold) {
      // Price moving up — buy
      await executeSwap(USDM, WETH, parseUnits('100', 18), 50);
    } else if (momentum < -this.threshold) {
      // Price moving down — sell
      await executeSwap(WETH, USDM, parseUnits('0.05', 18), 50);
    }
  }
}
```

### 4. Position management with stop-loss

```typescript
interface Position {
  token: string;
  entryPrice: number;
  amount: bigint;
  stopLoss: number;  // -2% default
  takeProfit: number; // +5% default
}

async function managePosition(position: Position, currentPrice: number) {
  const pnlPct = (currentPrice - position.entryPrice) / position.entryPrice;

  if (pnlPct <= position.stopLoss) {
    // Stop loss hit — exit immediately
    await executeSwap(position.token, USDM, position.amount, 100);
    console.log(`Stop loss triggered at ${(pnlPct * 100).toFixed(2)}%`);
    return 'closed';
  }

  if (pnlPct >= position.takeProfit) {
    // Take profit — exit
    await executeSwap(position.token, USDM, position.amount, 50);
    console.log(`Take profit at ${(pnlPct * 100).toFixed(2)}%`);
    return 'closed';
  }

  return 'open';
}
```

## Addresses

| Contract | Address | Notes |
|----------|---------|-------|
| Kumbaya SwapRouter | Check [kumbaya-dex-skill](../kumbaya-dex.md) | Uniswap V3 SwapRouter fork |
| USDm | Check MegaETH docs | Native stablecoin (18 decimals) |
| WETH | Check MegaETH docs | Wrapped ETH |

## Safety Rules

1. **Always set slippage protection** — never swap with `amountOutMinimum: 0`
2. **Always set deadline** — prevent stale transactions from executing
3. **Start with small amounts** — test on testnet first, then mainnet with minimal capital
4. **Monitor gas costs** — even with low MegaETH gas, high-frequency bots accumulate costs
5. **Handle reverts gracefully** — DEX swaps can fail due to insufficient liquidity
6. **Never hardcode private keys** — use environment variables or hardware wallets

## Related Skills

- [kumbaya-dex-skill](../kumbaya-dex.md) — DEX integration fundamentals
- [x402-payments-skill](../x402-payments.md) — Payment infrastructure
- [megaeth-dev-skill](https://github.com/0xBreadguy/megaeth-ai-developer-skills) — General MegaETH development
