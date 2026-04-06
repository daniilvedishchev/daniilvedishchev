<img width="6336" height="1584" alt="octurn_linkedin_banner_4x" src="https://github.com/user-attachments/assets/ed5c7550-cb2d-4317-9bf1-a0b84e768aff" />

<p align="center">
  <img src="https://github.com/user-attachments/assets/ed5c7550-cb2d-4317-9bf1-a0b84e768aff" alt="Octurn" width="100%">
</p>
# Octurn

**A C++20 strategy DSL and backtesting engine for quantitative trading research.**

Octurn lets you define trading strategies in a clean, declarative language — then parses, interprets, and evaluates them against real market data through a native C++ engine.

```
strategy SimpleMA {
  parameters {
    fast_ma: 5
    slow_ma: 30
  }
  indicators {
    RSI1 = RSI(AAPL_close, 12)
  }
  entry {
    when RSI1 > 50
  }
  exit {
    when RSI1 > 250
  }
}
```

---

## What it does

Octurn takes a strategy script → lexes → parses → builds an AST → interprets it against market data → outputs runtime results (signals, indicator values, trade flags).

The full pipeline:

```
.oct script → Lexer → Parser → AST → Interpreter → Runtime evaluation
                                                        ↓
                                            Polygon.io OHLC data
                                                        ↓
                                          Indicators / Signals / Trades
```

**Input:** a `.oct` strategy script + Polygon.io API key

**Output:** evaluated signals, indicator series, entry/exit flags, and backtest results

---

## DSL syntax

A strategy file has three top-level blocks:

```
config {
  equity: 100
  positionSize: 1
  slippageBps: 10
}

data [
  { ticker: AAPL timespan: day multiplier: 1 from: 2025-09-01 to: 2025-10-27 },
  { ticker: MSFT timespan: day multiplier: 1 from: 2025-08-01 to: 2025-10-27 }
]

strategy SimpleMA {
  parameters { ... }
  indicators { ... }
  entry { when <condition> }
  exit  { when <condition> }
}
```

- **`config`** — equity, position sizing, slippage
- **`data`** — tickers, timeframes, date ranges (fetched via Polygon.io)
- **`strategy`** — parameters, indicators (RSI, MA, etc.), entry/exit rules with boolean conditions

---

## Architecture

```
Octurn/
├── lexer/          # Tokenizer — breaks .oct scripts into tokens
├── parser/         # Recursive descent parser → AST
├── node/           # AST node types (expressions, params, indicators, rules)
├── interpreter/    # Evaluates AST over market data, resolves indicators & signals
├── engine/         # Top-level orchestrator — ties lexer→parser→interpreter→data
├── ta/             # Technical analysis library (RSI, MA, extensible)
├── backtester/     # Backtest loop — applies signals to price data, tracks P&L
├── execution/      # Execution engine — order generation from signals
├── trade/          # Trade type definitions and tracking
├── config/         # Config parsing, validation rules, slippage tables
├── mappers/        # Maps raw API responses → internal market data types
├── dataLayer/      # Market data view abstraction
├── src/polygon/    # Polygon.io REST client + data feed integration
├── injector/       # Dependency injection utilities
├── log/            # Structured logging
├── utils/          # Shared helpers
└── types/          # Core type definitions (OHLC, bars, series)
```

---

## How it works

### 1. Lexer
Tokenizes the `.oct` script into a stream of typed tokens (keywords, identifiers, literals, operators, delimiters).

### 2. Parser
Recursive descent parser that consumes the token stream and produces a structured AST. Handles `config`, `data`, and `strategy` blocks with nested `parameters`, `indicators`, `entry`, and `exit` sub-blocks.

### 3. Interpreter
Walks the AST and evaluates it:
- Resolves `data` blocks → fetches OHLC bars from Polygon.io
- Computes `indicators` → dispatches to the TA library (RSI, MA, etc.)
- Evaluates `entry` / `exit` conditions → produces boolean signal arrays
- Manages execution state: `parse → wait_for_data → ready → run`

### 4. Backtester
Takes the signal arrays + price data and simulates execution: tracks positions, applies slippage, computes equity curve and trade log.

### 5. Execution engine
Translates signals into order-level events. Designed for future connection to live execution.

---

## Runtime output

```
Octurn Runtime
==============
Config
------
equity        : 100
positionSize  : 1
slippageBps   : 10

Data Sources
------------
- AAPL | day | 1x | 2025-09-01 → 2025-10-27 | 40 bars
- MSFT | day | 1x | 2025-08-01 → 2025-10-27 | 61 bars

Indicators
----------
RSI1          : length = 40
RSI1 tail     : [49.03, 51.62, 48.39, 56.07, 67.06, 67.53, 59.90, 61.15, 64.61, 70.00]

Signals
-------
Entry true at : [12..27, 31, 33..39]    (23 signals)
Exit true at  : []                       (0 signals)
```

---

## Tech stack

| Component | Technology |
|---|---|
| Language | C++20 |
| Build | CMake |
| Market data | Polygon.io REST API (via [cpr](https://github.com/libcpr/cpr)) |
| JSON | [nlohmann/json](https://github.com/nlohmann/json) |
| Technical analysis | Custom C++ (RSI, MA — extensible) |

---

## Build

```bash
mkdir build && cd build
cmake ..
cmake --build .
```

Requires `cpr` and `nlohmann_json` (install via vcpkg, Homebrew, or system package manager).

---

## Run

```cpp
#include "engine/octurn.hpp"

int main() {
    std::string script = R"(
        config { equity: 100 positionSize: 1 slippageBps: 10 }
        data [
            { ticker: AAPL timespan: day multiplier: 1
              from: 2025-09-01 to: 2025-10-27 }
        ]
        strategy MyStrat {
            parameters { period: 14 }
            indicators { RSI1 = RSI(AAPL_close, 12) }
            entry { when RSI1 > 50 }
            exit  { when RSI1 > 80 }
        }
    )";

    octurn engine(script, "YOUR_POLYGON_API_KEY");
    engine.run();
}
```

---

## Roadmap

- [ ] Richer DSL syntax (multi-condition logic, cross-asset rules)
- [ ] More indicators (EMA, MACD, Bollinger, ATR)
- [ ] Portfolio-level logic (multi-strategy, allocation)
- [ ] Backtest metrics (Sharpe, drawdown, profit factor)
- [ ] JSON export for runtime results
- [ ] Validation & diagnostic error messages
- [ ] Execution bridge (paper → live)
- [ ] WASM compilation for web integration

---

## Status

Early-stage — the core pipeline works end-to-end (script → parse → fetch data → compute indicators → generate signals → backtest). Actively being extended with richer DSL features and backtest analytics.

---

## License

MIT

---

## Contact

Open an issue or reach out if you're interested in strategy infrastructure, DSL design, or quant tooling.
