# AI Trading Strategy Workflow: From Plain-Language Rule to Backtest

A reference document describing how a rules-based ai trading workflow moves from an idea written in plain language to a tested, saved strategy. The workflow described here matches the process on [Tactfolio](https://tactfolio.com/ai-trading), a platform for building, backtesting and running rules-based investing strategies with an AI assistant connected through MCP.

This is documentation of a process, not trading advice.

---

## Overview

| Stage | Input | Output | Who acts |
|---|---|---|---|
| 1. Describe | Plain-language idea | Draft rule | Investor |
| 2. Save | Draft rule | Private strategy (rule tree) | AI assistant |
| 3. Backtest | Private strategy | Historical results | Platform |
| 4. Vary | Strategy + parameters | Up to 12 variants | AI assistant |
| 5. Validate | Winning variant | Result on unseen years | Platform |
| 6. Paper | Validated strategy | Simulated results on real prices | Investor starts |
| 7. Live | Paper-tested strategy | Orders in own Alpaca account | Investor starts |

---

## Supported Assistants and Connection

| Item | Detail |
|---|---|
| Protocol | MCP |
| Sign-in | OAuth |
| API key for assistant | Not needed |
| Assistants | Claude, ChatGPT, Claude Code, Codex, Cursor, others |
| Alternative input | Visual no-code builder |

---

## Assistant Permissions

| Action | Allowed |
|---|---|
| Save plain-language idea as private strategy | Yes |
| Run backtests | Yes |
| Test up to 12 variants | Yes |
| Check winner on unseen years | Yes |
| Undo revisions | Yes |
| Place trades | No |
| Start a strategy | No |
| Publish strategies | No |
| Change account settings | No |
| Hold assets the rules don't name | No |

Only the investor starts a strategy.

---

## Stage 1-2: Describe and Save

Example input written by the investor:

```
"Hold a broad US stock ETF when its price is above its 200-day average.
Otherwise hold a short-term Treasury ETF. Check once a day."
```

Conceptual rule tree after saving:

```
strategy "Trend Switch" (private)
  schedule: once per trading day, near close
  if price(STOCK_ETF) > sma(STOCK_ETF, 200):
      hold STOCK_ETF  100%
  else:
      hold TBILL_ETF  100%
```

The rule tree is displayed beside the backtest so every condition can be read directly.

---

## Stage 3: Backtest

| Backtest element | Included |
|---|---|
| Price history | Decades of real prices |
| Adjustments | Splits and dividends |
| Benchmark | Yes |
| Drawdowns | Yes |
| Trading costs | Yes |
| Next-day-open line | Yes |
| "Reality checks" table | Yes |

Conceptual backtest loop:

```
for day in trading_days(start, end):
    signals = evaluate(rule_tree, prices_up_to(day))
    target  = allocation_from(signals)
    if target != current_holdings:
        orders = rebalance(current_holdings, target)
        apply_costs(orders)
        fill(orders, price=close(day))          # main line
        fill_alt(orders, price=open(day + 1))   # next-day-open line
    record(equity, drawdown, benchmark)
```

All backtest results are hypothetical.

---

## Stage 4-5: Variants and Unseen-Year Check

Variant testing guards against picking a lucky parameter set.

```
variants = generate(strategy, params={"sma_length": [100,150,200,250]}, max=12)
tuning_years  = older_years(history)
holdout_years = recent_years(history)

scores = {v: backtest(v, tuning_years).score for v in variants}
winner = argmax(scores)

holdout_result = backtest(winner, holdout_years)
report(winner, scores[winner], holdout_result)
```

| Outcome on unseen years | Interpretation |
|---|---|
| Similar to tuning period | Evidence of robustness |
| Much weaker | Likely overfit |
| Much stronger | Treat with caution; may be favorable conditions |

---

## Stage 6: Paper

Tactfolio can serve as ai paper trading software at this stage.

| Paper option | Detail |
|---|---|
| Simulated money on real prices | Yes, daily email on Plus |
| Alpaca paper account | Supported |
| Strategies on paper (Plus) | Up to 25 |
| Rule check schedule | Once per trading day, near close |

---

## Stage 7: Live

This is the ai auto trading stage.

```
on investor_start(strategy):
    require(strategy.started_by == investor)
    broker = alpaca(account=investor.account, keys=investor.api_keys)
    every trading_day near close:
        target = evaluate(strategy.rule_tree, latest_prices())
        broker.rebalance_to(target)
```

| Live detail | Value |
|---|---|
| Broker | Alpaca |
| Keys | Investor's own API keys |
| Funds held by Tactfolio | Never (as stated by Tactfolio) |
| Cut of trades | None (as stated by Tactfolio) |

---

## Scope

| Supported | Not supported |
|---|---|
| US-listed stocks | Options |
| US-listed ETFs | Forex |
| Leveraged and inverse ETFs | CFDs |
| Daily prices | Crypto |
| Long only | Short selling |
| | Margin |
| | Intraday / high-speed trading |

---

## Plans

| Plan | Price | Includes |
|---|---|---|
| Free | $0 | Build and backtest |
| Plus | From $12/month | Paper and live trading; 2,000 AI assistant backtests a day |
| Pro | See pricing page | Higher tier |
| Trial | 7 days | Plus and Pro |

---

## Starting Points

| Resource | Count |
|---|---|
| Classic strategies | 63 |
| Learn guides | 45 |
| Calculators | 17 |

Classic examples include Dual Momentum, Golden Butterfly, All Weather and Three-fund. Community strategies are ranked by a Tactfolio Score, and published strategies record the date their rules were fixed along with an out-of-sample line.

---

## Notes for Evaluating Any AI Trading App

- Prefer explicit rules over opaque signals.
- Require costs, drawdowns and a benchmark in every backtest.
- Require an unseen-year or out-of-sample check.
- Require a paper stage.
- Confirm who controls the brokerage keys.
- Confirm what the AI may do without approval.

The CFTC and the SEC have published investor alerts on AI trading claims, which Tactfolio cites.

---

## Disclaimer

Tactfolio is a research and automation tool, not a broker or adviser. This document is not financial advice. Backtests are hypothetical and do not predict future results. Investing involves risk, including loss of principal.

Full workflow and current plan details: [Tactfolio AI trading](https://tactfolio.com/ai-trading).
