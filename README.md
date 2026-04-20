<p align="center">
  <img src="https://img.shields.io/badge/C%2B%2B-20-blue?style=flat-square&logo=cplusplus&logoColor=white" alt="C++20"/>
  <img src="https://img.shields.io/badge/build-CMake-064F8C?style=flat-square&logo=cmake&logoColor=white" alt="CMake"/>
  <img src="https://img.shields.io/badge/data-Polygon.io-7C3AED?style=flat-square" alt="Polygon.io"/>
  <img src="https://img.shields.io/badge/license-MIT-green?style=flat-square" alt="MIT"/>
</p>

# Octurn

**A C++20 strategy engine and domain-specific language for quantitative trading.**

Octurn lets you write a complete trading strategy in a few lines of declarative code — no boilerplate, no framework, no Python notebooks. Define your market, indicators, entry/exit logic, and risk parameters in a single `.oct` file. The engine handles data ingestion, signal generation, execution simulation, and P&L tracking.

```
strategy "BTC EMA Cross"
  mode backtest
  market BTCUSDT 1m
  from 2025-01-01 to 2025-06-30

capital 2000
risk 0.25%

execution market
fill full
slippage base

fast = ema(8)
slow = ema(21)

buy when fast > slow
close when fast < slow

stop 0.8%
take 1.6%
```

That's a complete, runnable strategy. 18 lines.

---

## Why Octurn

Most quant workflows are fragmented — strategy logic in notebooks, indicators hardcoded in scripts, backtesting in one stack, execution in another. Octurn collapses this into a single pipeline:

**One file defines the strategy.** The same `.oct` file runs in backtest and live mode — change one keyword.

**The engine is the infrastructure.** Lexer → Parser → AST → Interpreter → Backtester. Hand-written in C++20, no external DSL tooling.

**Execution is realistic.** Square-root market impact model, FOK/GTC fill modes, commission tracking, partial fills across bars. No fantasy P&L.

---

## Architecture

```
┌─────────────┐     ┌─────────┐     ┌─────────┐     ┌─────────────┐     ┌────────────┐
│  .oct file  │────▶│  Lexer  │────▶│ Parser  │────▶│ Interpreter │────▶│ Backtester │
└─────────────┘     └─────────┘     └─────────┘     └─────────────┘     └────────────┘
                                                            │                   │
                                                    ┌───────▼──────┐    ┌───────▼───────┐
                                                    │  DataLayer   │    │   Execution   │
                                                    │ (Polygon.io) │    │    Engine     │
                                                    └──────────────┘    └───────────────┘
```

| Module | Path | Role |
|--------|------|------|
| **DSL Lexer** | `dsl/lexer/` | Tokenizes `.oct` scripts — keywords, dates, timeframes, percentages, operators |
| **DSL Tokens** | `dsl/token/` | Token types enum (30+ types) and keyword map |
| **Parser** | `parser/` | Recursive-descent parser → AST |
| **Interpreter** | `interpreter/` | Evaluates AST, computes indicators, generates entry/exit signals |
| **DataLayer** | `dataLayer/` | Unified market data access with O(1) timestamp lookup |
| **DataInjector** | `injector/` | Fetches OHLCV bars from Polygon.io REST API |
| **ExecutionEngine** | `execution/` | Order filling — FOK, GTC, slippage, position sizing, VWAP |
| **Backtester** | `backtester/` | Bar-by-bar simulation, stop-loss tracking, mark-to-market |
| **Config** | `config/` | Rule-based config validation with lambda validators |
| **TA Library** | `ta/` | Technical indicators — SMA, RSI (Wilder's smoothing) |
| **Account** | `account/` | Equity, free cash, margin, realized/unrealized P&L |
| **Trade** | `trade/` | Trade struct — qty state, price state, fill log, status |

---

## DSL Reference

### Strategy header

```
strategy "name"
  mode backtest          // backtest | live
  market BTCUSDT 1m      // symbol + timeframe (1m, 5m, 15m, 1h, 4h, 1d, 1w)
  from 2025-01-01 to 2025-12-31
```

### Capital & risk

```
capital 2000             // starting balance (quote currency)
risk 0.25%               // max loss per trade (% of capital)
```

Position size is calculated automatically:  
`position = (capital × risk%) / (entry × stop%)`

### Execution

```
execution market         // market | limit
fill full                // full | fok | gtc
slippage base            // optimistic | base | pessimistic
```

### Indicators

```
fast = ema(8)
slow = ema(21)
strength = rsi(14)
volatility = atr(14)
trend = ema(50) - ema(200)
```

Available: `ema`, `sma`, `rsi`, `atr`, `macd`, `bollinger`, `stdev`, `highest`, `lowest`

### Rules

```
buy when fast > slow
close when fast < slow

// multi-condition
buy when fast > slow and strength < 40
```

Operators: `>` `<` `>=` `<=` `==` `!=` `and` `or`

### Risk management

```
stop 0.8%                // stop-loss (% from entry)
take 1.6%                // take-profit (% from entry)
```

### Session filter

```
session london           // london | new-york | tokyo | all
```

---

## Execution Model

The execution engine simulates realistic order fills:

**Position sizing** — Risk-based. Each trade risks exactly `risk%` of current capital. The stop-loss distance determines position size.

**Market impact** — Square-root participation model:

```
impact_bps = impact_coef × √(order_qty / bar_volume)
fill_price = open × (1 ± spread ± impact)
```

**Fill modes:**
- **FOK** (fill-or-kill) — entire order fills on one bar or gets rejected
- **GTC** (good-til-cancelled) — partial fills across bars with VWAP tracking

**Slippage presets:**

| Preset | Max Participation | Impact Coef | Use Case |
|--------|:-:|:-:|----------|
| `optimistic` | 10% | 10.0 | Liquid markets, small orders |
| `base` | 5% | 20.0 | Default — realistic for most cases |
| `pessimistic` | 2% | 30.0 | Thin markets, stress testing |

---

## Data Layer

Market data flows through a typed pipeline:

1. **Polygon.io API** → HTTP fetch via `cpr` → JSON response
2. **DataInjector** → Parses `results[]` → move-constructs `Equity` objects
3. **EquityMap** → `unordered_map<string, Equity>` with O(1) timestamp index
4. **DataLayer** → Unified access via `getBar(ticker, idx)` and `getValue(key, idx)`

Each `Equity` stores OHLCV vectors + a `timestampIndexMap` for constant-time lookups.

---

## Build

**Dependencies:** CMake ≥ 3.14, C++20 compiler, [cpr](https://github.com/libcpr/cpr), [nlohmann/json](https://github.com/nlohmann/json)

```bash
mkdir build && cd build
cmake ..
cmake --build .
```

**Run:**

```bash
./Octurn
```

---

## Project Structure

```
Octurn/
├── main.cpp              # Entry point
├── CMakeLists.txt
├── engine/               # Pipeline orchestrator (Lexer → Parser → Interpreter)
├── dsl/
│   ├── lexer/            # Hand-written tokenizer
│   ├── token/            # Token types + keyword map
│   ├── fields/           # Enum definitions (mode, session, execution)
│   └── validator/        # DSL validation rules
├── parser/               # Recursive-descent parser → AST
├── interpreter/          # AST evaluator + signal generator
├── backtester/           # Bar-by-bar simulation engine
├── execution/            # Order execution (FOK, GTC, slippage)
├── dataLayer/            # Market data access layer
├── injector/             # Polygon.io data fetcher
├── ta/                   # Technical analysis (MA, RSI)
├── config/               # Config rules, validation, slippage tables
├── account/              # Equity, margin, P&L tracking
├── trade/                # Trade struct + lifecycle
├── equity/               # OHLCV storage + timestamp index
├── marketTypes/          # Bar, QtyState, PriceState, enums
├── types/                # Core type system (AnyValue variant)
├── mappers/              # Function registry (TA dispatch)
├── log/                  # File logger
├── utils/                # Debug printing utilities
├── src/polygon/          # Polygon.io REST client
└── api/                  # REST API layer (RabbitMQ integration)
```

---

## Roadmap

- [ ] Walk-forward validation
- [ ] More indicators (EMA, Bollinger, MACD, ATR in DSL)
- [ ] Portfolio-level logic (multi-asset, correlation)
- [ ] JSON/CSV trade export
- [ ] Web dashboard (WASM frontend)
- [ ] Execution bridge to live brokers
- [ ] Strategy optimization / parameter sweep

---

## Documentation

📄 **[Octurn DSL User Guide (PDF)](docs/octurn_dsl_guide.pdf)** — complete reference for writing strategies

---

## Contact

Open an issue or reach out if you're interested in the engine, strategy infrastructure, or building on top of Octurn.
