# Daniil Vedishchev

Quant-oriented engineering student (France) with a finance background and strong software fundamentals.  
I build research tooling that connects **market data → strategy definition → compute/backtesting pipelines**.

**Currently building:** **Kayiros** (R&D backtesting platform) with **Octurn** — a custom **C++20 Strategy DSL**.

**Highlights**
- **Octurn DSL:** lexer/parser/AST + readable diagnostics for fast research iteration
- **C++ compute core:** indicator primitives (MA/RSI) + function dispatch callable from DSL scripts
- **Market data pipeline:** async OHLC fetch + engine integration designed for web tooling (WASM-friendly bridge)
- **Scaling scaffold:** queue-based job execution + HTTP service integration points for orchestration

## Featured: Kayiros (R&D Backtesting Platform) + Octurn (Strategy DSL)

**Kayiros** is an R&D backtesting platform built to speed up strategy research.  
It combines a **custom strategy DSL (Octurn)** with a **C++ compute core**, integrates with **web tooling (WASM)**, and is structured for **scalable job execution patterns**.

At the core is **Octurn**, a C++20 trading-strategy DSL and backtesting engine:  
➡️ **Octurn repo:** https://github.com/daniilvedishchev/Octurn

### What I built (high level)
- **Octurn — custom strategy DSL**
  - Lexer, parser, AST, and readable diagnostics
  - Block structure for research workflows: `config / data / indicators / entry / exit`
- **Runtime / interpreter**
  - Explicit execution states (parse → wait for data → ready → run) for predictable evaluation
- **C++ compute core**
  - Indicator primitives (e.g., MA, RSI) + function dispatch so indicators are callable from DSL scripts
- **Market data ingestion**
  - Async OHLC fetch and transfer into the C++ engine via a **WASM bridge** (callbacks, readiness flags, queued results, cleanup)
- **Service + scaling scaffolding (prototype)**
  - Queue publisher/worker skeleton for backtest jobs (RabbitMQ pattern)
  - HTTP service scaffold for endpoints + future orchestration hooks (Drogon)
- **UI scaffold**
  - React/Vite setup for a backtest “cockpit” (future dashboards, run history, parameter sweeps)
- **Quant framing (roadmap-ready)**
  - Prepared for transaction costs/slippage, reproducibility controls, and out-of-sample workflows without changing the DSL surface

## Badges
![C++](https://img.shields.io/badge/C%2B%2B-20-blue)
![Python](https://img.shields.io/badge/Python-3.x-blue)
![TypeScript](https://img.shields.io/badge/TypeScript-ready-blue)
![WebAssembly](https://img.shields.io/badge/WebAssembly-WASM-654ff0)
![React](https://img.shields.io/badge/React-Vite-61dafb)
![Docker](https://img.shields.io/badge/Docker-ready-2496ed)
![RabbitMQ](https://img.shields.io/badge/RabbitMQ-queues-ff6600)
![Quant](https://img.shields.io/badge/Quant-backtesting-1f4e79)

## Tiny Octurn example
```octurn
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

