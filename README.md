# Daniil Vedishchev

Quant-oriented engineering student (France) with a finance background and strong software fundamentals.  
I build research tooling that connects **market data → strategy definition → compute/backtesting pipelines**.

**Currently building:** **Kayiros** (R&D backtesting platform) with **Octurn** — a custom **C++20 Strategy DSL**.

**Highlights**
- **Octurn DSL:** lexer/parser/AST + readable diagnostics for fast research iteration
- **C++ compute core:** indicator primitives (MA/RSI) + function dispatch callable from DSL scripts
- **WASM-ready data flow:** async OHLC ingestion and a bridge pattern suitable for browser runtimes
- **Scaling scaffold:** queue-based job execution + HTTP service integration points for orchestration

## Featured: Kayiros (R&D Backtesting Platform) + Octurn (Strategy DSL)

**Kayiros** is my R&D workspace for systematic trading research — designed for fast iteration, reproducibility, and scaling from local runs to queued execution.

At the core is **Octurn**, a C++20 trading-strategy DSL and backtesting engine:  
➡️ **Octurn repo:** https://github.com/daniilvedishchev/Octurn

### What I built (high level)
- **Octurn — custom strategy DSL**
  - Lexer, parser, AST, and readable diagnostics
  - Block structure for research workflows: `data / parameters / indicators / entry / exit`
- **Runtime / interpreter**
  - Explicit execution states (parse → wait for data → ready → run) for predictable evaluation
- **C++ compute core**
  - Indicator primitives (e.g., MA, RSI) + function dispatch so indicators are callable from DSL scripts
- **Data ingestion + WASM-friendly flow**
  - Async OHLC fetching and a bridge pattern suitable for browser/WASM runtimes
- **Scaling scaffolding (prototype)**
  - Queue-based job execution patterns (publisher/worker)
  - HTTP service integration points (Drogon-style scaffold)
- **UI scaffold**
  - “Backtest cockpit” idea (run history, parameter sweeps, analytics dashboards)

## Tech Stack

**Core / Engine**
- C++20, CMake
- Eigen (linear algebra), nlohmann/json
- HTTP client: cpr

**DSL**
- Custom lexer/parser + AST
- Interpreter/runtime with staged evaluation

**Web / UI**
- JavaScript / TypeScript
- React + Vite
- Emscripten / WebAssembly bridge

**Data & APIs**
- OHLC pipelines
- Polygon API (market data)

**Infra / Scaling (prototype)**
- RabbitMQ (publisher/worker pattern)
- Drogon (HTTP service scaffold)
- Docker

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
