# Daniil Vedishchev

Quant-oriented engineering student (France) with a finance background and strong software fundamentals.  
I build research tooling that connects **market data → strategy definition → compute/backtesting pipelines**.

## Featured: Kayiros (R&D Backtesting Platform) + Octurn (Strategy DSL)

**Kayiros** is my R&D workspace for systematic trading research — designed for fast iteration, reproducibility, and scaling from local runs to queued execution.

At the core is **Octurn**, a C++20 trading-strategy DSL and backtesting engine:  
➡️ **Octurn repo:** https://github.com/daniilvedishchev/Octurn

### What I built (high level)
- **Octurn — custom strategy DSL**
  - Lexer, parser, AST, and readable diagnostics
  - Block structure built for research workflows: `data / parameters / indicators / entry / exit`
- **Runtime / interpreter**
  - Explicit execution states (parse → wait for data → ready → run) for predictable evaluation
- **C++ compute core**
  - Indicator primitives (e.g., MA, RSI) + function dispatch so indicators are callable from DSL scripts
- **Data ingestion + WASM-friendly flow**
  - Async OHLC fetching and a bridge pattern suitable for browser/WASM runtimes
- **Scaling scaffolding (prototype)**
  - Queue-based job execution patterns (RabbitMQ-style publisher/worker)
  - HTTP service integration points (Drogon-style scaffold)
- **UI scaffold**
  - “Backtest cockpit” idea (run history, parameter sweeps, analytics dashboards)

### Tiny Octurn example
```octurn
data [
  { ticker: AAPL timespan: day multiplier: 1 from: 2024-01-01 to: 2024-03-01 }
]

strategy simple_ma {
  parameters { fast: 5 slow: 30 }
  indicators {
    fast_ma = MA(AAPL_close, fast)
    slow_ma = MA(AAPL_close, slow)
  }
  entry { when fast_ma > slow_ma }
  exit  { when fast_ma < slow_ma }
}
