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

### 5. Mean-reversion strategy (alternative to momentum)

Mean-reversion bets that price returns to the mean after extending. Works well in ranging markets where momentum bots get chopped up.

```typescript
class MeanReversionStrategy {
  private prices: PricePoint[] = [];
  private readonly windowSize = 100; // 100 blocks = 1s window
  private readonly zEntry = 2.0;     // enter when price > 2 std devs from mean
  private readonly zExit  = 0.5;     // exit when price returns within 0.5 std devs

  private mean(arr: number[]): number {
    return arr.reduce((a, b) => a + b, 0) / arr.length;
  }
  private std(arr: number[], m: number): number {
    return Math.sqrt(arr.reduce((s, x) => s + (x - m) ** 2, 0) / arr.length);
  }

  async onPriceUpdate(price: number, timestamp: number) {
    this.prices.push({ price, timestamp });
    if (this.prices.length > this.windowSize) this.prices.shift();
    if (this.prices.length < this.windowSize) return;

    const arr = this.prices.map(p => p.price);
    const m = this.mean(arr);
    const s = this.std(arr, m);
    if (s === 0) return;
    const z = (price - m) / s;

    if (z > this.zEntry)  await this.short();   // overbought → fade
    if (z < -this.zEntry) await this.long();    // oversold → buy
    if (Math.abs(z) < this.zExit) await this.flatten(); // back to mean → exit
  }
  // short(), long(), flatten() wrap executeSwap()
}
```

## Backtesting before mainnet

**Never deploy a strategy live without backtesting it on historical data first.** A strategy that looks good on a chart can lose money in seconds when run against real market microstructure.

### Minimum viable backtest

```typescript
import { readFileSync } from 'fs';

interface Bar { timestamp: number; price: number; }

function backtest(strategy: { onPriceUpdate: (p: number, t: number) => Promise<void> }, bars: Bar[]) {
  let trades: { side: 'BUY' | 'SELL'; price: number; ts: number }[] = [];

  // Inject a recording wrapper so executeSwap pushes to `trades` instead of hitting chain
  globalThis.executeSwap = async (tokenIn, tokenOut, amount, slippage) => {
    const side = tokenIn === USDM ? 'BUY' : 'SELL';
    trades.push({ side, price: bars[bars.length - 1].price, ts: bars[bars.length - 1].timestamp });
  };

  for (const bar of bars) {
    strategy.onPriceUpdate(bar.price, bar.timestamp);
  }
  return computeStats(trades);
}

function computeStats(trades: typeof trades) {
  // Pair BUY → next SELL, compute P&L, win rate, max drawdown
  // ...
}
```

### Critical backtest considerations

1. **Model fees and slippage realistically** — assuming 0 slippage will inflate every backtest. Use the worst-case spread observed for the pool, not the median.
2. **No look-ahead bias** — the strategy must only see prices at-or-before the current bar's timestamp. `eth_sendRawTransactionSync` is fast on MegaETH but it is NOT instantaneous to *price discovery* — your bot reacts to the current block, not the next one.
3. **Test in-sample AND out-of-sample** — split historical data into a tuning window (60-70%) and a hold-out window (30-40%). If out-of-sample WR collapses, the strategy is curve-fit.
4. **Walk-forward validation** — slice data into rolling windows, re-tune parameters per window, evaluate on the next window. If the edge doesn't persist across windows, don't deploy.
5. **Always run a paper-trading mode first** — wrap `executeSwap` to log intended trades without sending them. Compare paper-trade P&L to backtest expectation for at least a week before going live.

## Production reliability

A 10ms chain doesn't help if your bot is offline. Build for the failure cases.

### WebSocket reconnect with exponential backoff

```typescript
function createResilientClient() {
  let attempts = 0;
  const connect = () => {
    const transport = webSocket('wss://rpc.megaeth.com/ws', {
      reconnect: { attempts: Infinity, delay: () => Math.min(1000 * 2 ** attempts++, 30000) },
      keepAlive: { interval: 5000 },
    });
    transport.value?.subscribe?.('disconnect', () => console.warn('WS disconnect — backing off'));
    transport.value?.subscribe?.('reconnect', () => { attempts = 0; console.log('WS reconnected'); });
    return createPublicClient({ chain: megaeth, transport });
  };
  return connect();
}
```

### Missed-block / watchdog pattern

If no new block arrives within ~50ms, assume the WebSocket is stale and reconnect. On MegaETH, going 100ms+ without a block is abnormal.

```typescript
let lastBlockTime = Date.now();
client.watchBlocks({ onBlock: () => { lastBlockTime = Date.now(); } });

setInterval(() => {
  if (Date.now() - lastBlockTime > 500) {
    console.warn('Stale connection — reconnecting');
    // tear down and rebuild client
  }
}, 200);
```

### Kill switch

Always have a way to halt all trading instantly. Read a flag from a file, environment variable, or a contract — and check it before every trade.

```typescript
async function shouldTrade(): Promise<boolean> {
  if (process.env.KILL_SWITCH === '1') return false;
  if (existsSync('/tmp/kill_switch')) return false;
  return true;
}
```

## MEV and front-running awareness

10ms blocks don't eliminate MEV — they reshape it. Things to know:

- **Sandwich attacks**: a searcher can frontrun a large swap (in the same block), then backrun. Mitigation: use private RPCs (when available), set tight `amountOutMinimum`, and split large orders.
- **Same-block races**: if your bot and another bot both react to the same price move, the one with the faster RPC and lower latency wins. Co-locating with an RPC node, or using `eth_sendRawTransactionSync` to skip mempool, both help.
- **Avoid mempool exposure for sensitive trades**: prefer `eth_sendRawTransactionSync` over standard `eth_sendRawTransaction` when racing — your transaction lands in the next block without sitting in the public mempool.
- **Pool snipers**: when new pools launch, snipers fire identical bots. Don't assume your edge survives if 50 other bots have the same idea.

## Addresses

| Contract | Address | Notes |
|----------|---------|-------|
| Kumbaya SwapRouter | Check [kumbaya-dex-skill](../kumbaya-dex.md) | Uniswap V3 SwapRouter fork |
| USDm | Check MegaETH docs | Native stablecoin (18 decimals) |
| WETH | Check MegaETH docs | Wrapped ETH |

## Safety Rules

1. **Backtest before deploying live** — see the Backtesting section. A strategy that hasn't been validated on historical data should not touch real money.
2. **Paper-trade for at least a week** — log intended trades without sending them. Reconcile against your backtest expectation before committing capital.
3. **Always set slippage protection** — never swap with `amountOutMinimum: 0`
4. **Always set deadline** — prevent stale transactions from executing
5. **Start with small amounts** — test on testnet first, then mainnet with minimal capital
6. **Monitor gas costs** — even with low MegaETH gas, high-frequency bots accumulate costs
7. **Handle reverts gracefully** — DEX swaps can fail due to insufficient liquidity
8. **Build a kill switch** — every bot needs an instant-halt mechanism (file flag, env var, or on-chain pause)
9. **Never hardcode private keys** — use environment variables or hardware wallets
10. **Set a daily loss limit** — circuit-break the bot if cumulative loss exceeds X% in 24h, regardless of strategy signal

## Related Skills

- [kumbaya-dex-skill](../kumbaya-dex.md) — DEX integration fundamentals
- [x402-payments-skill](../x402-payments.md) — Payment infrastructure
- [megaeth-dev-skill](https://github.com/0xBreadguy/megaeth-ai-developer-skills) — General MegaETH development
