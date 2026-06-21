# Agent Notes: project-degen-bot

## What This Repo Is

`project-degen-bot` is a fork of `BowTiedDevil/degenbot`. The upstream project is a Python 3.12 toolkit for building EVM trading and DeFi automation around liquidity pools. It models ERC-20 tokens, pool state, swap math, route profitability, local chain forks, database-backed pool metadata, and protocol-specific behavior for Uniswap, Curve, Aerodrome, PancakeSwap, SushiSwap, Balancer, Camelot, SwapBased, Chainlink, and Aave V3.

For the current business goal, treat this repo as the research and calculation layer for a future Curve Finance arbitrage system. It helps answer: "Given current pool state, is this route profitable and what swap amounts should be used?" It is not yet a complete deployed bot that borrows a flash loan, submits protected transactions, manages private keys, scans the mempool, or runs unattended on a VPS.

## Goal Context

- Target: build toward Curve Finance arbitrage with flash loans.
- Monthly target: 30,000 JMD to cover subscriptions.
- Practical implication: every opportunity must clear gas, flash-loan fees, slippage, failed transaction risk, RPC latency, and MEV competition before it counts as profit.
- Development implication: start with offline/mainnet-fork simulation and paper calculations before any production key or funded wallet is attached.

## Top-Level Folder Map

- `.codex/` - local documentation and operating notes for this fork.
- `.github/` - upstream GitHub automation and issue/PR metadata.
- `.opencode/` - upstream agent/tooling configuration.
- `contract_reference/` - Solidity/Vyper contract snapshots used as protocol references, especially Aave.
- `debug/` - investigation notes for protocol edge cases and historical bugs.
- `docs/` - markdown documentation for config, CLI commands, Aave flows, and arbitrage internals.
- `rust/` - Rust extension source compiled through maturin as `degenbot_rs` for faster low-level EVM/math operations.
- `scripts/` - helper scripts for development/debug workflows.
- `src/degenbot/` - main Python package.
- `tests/` - Python and Rust integration tests with protocol fixtures.
- `AGENTS.md` - upstream development conventions and commands for agents.
- `justfile` - task runner for tests, linting, formatting, Rust builds, and docs.
- `pyproject.toml` - Python package metadata, dependencies, CLI entry point, lint/type/test configuration, and maturin build setup.
- `uv.lock` - locked dependency graph for uv-based installs.

## How The Main Package Fits Together

- `src/degenbot/__init__.py` exposes the public package surface: pool classes, arbitrage helpers, token helpers, registries, connection managers, Chainlink helpers, and Rust-backed functions.
- `connection/` owns Web3 provider registration and access. Most pool classes rely on the active connection manager or configured RPC endpoints.
- `config.py` creates and loads `~/.config/degenbot/config.toml`. RPC endpoints live under `[rpc]`, keyed by chain ID. The database defaults to `~/.config/degenbot/degenbot.db`.
- `erc20/`, `registry/`, and `checksum_cache.py` provide token objects, shared token/pool registries, and address normalization.
- `uniswap/`, `curve/`, `aerodrome/`, `pancakeswap/`, `sushiswap/`, `balancer/`, `camelot/`, and `swapbased/` implement protocol-specific pool state and swap math.
- `arbitrage/` contains the opportunity calculation helpers. For this goal, `UniswapCurveCycle` is the most directly relevant existing class.
- `aave/` contains Aave V3 state, event, math, position, and processor logic. Existing flash-loan material is primarily documentation and protocol analysis, not a finished flash-loan executor.
- `database/` and `migrations/` define SQLAlchemy models and Alembic migrations for storing token, pool, exchange, and Aave-derived data.
- `cli/` defines the `degenbot` Click CLI. Commands cover database work, exchange activation, pool updates, and Aave state workflows.
- `provider/`, `contract/`, `abi_adapter.py`, and `functions.py` wrap lower-level EVM call, ABI, and provider behavior.
- `anvil_fork.py` supports local forked-chain testing.

## Curve And Arbitrage Path

The existing Curve implementation centers on `src/degenbot/curve/curve_stableswap_liquidity_pool.py`.

- `CurveStableswapPool` models Curve V1 StableSwap pools.
- It fetches balances, fee parameters, amplification values, metapool/base-pool relationships, and token lists.
- It can estimate swap output with `calculate_tokens_out_from_tokens_in`.
- It keeps state in `CurveStableswapPoolState` and can update live state through `auto_update`.
- It publishes state updates to subscribed arbitrage helpers.

The existing mixed Uniswap/Curve arbitrage helper is `src/degenbot/arbitrage/uniswap_curve_cycle.py`.

- `UniswapCurveCycle` accepts an input token, a sequence of pools, an id, and an optional max input.
- It currently expects the Curve pool at position 1, meaning a typical path is `Uniswap pool -> Curve pool -> Uniswap pool`.
- `calculate()` checks for a minimum profitable exchange rate, optimizes the input amount, and returns an `ArbitrageCalculationResult`.
- Result data includes input token, profit token, input amount, profit amount, state block, and per-pool swap amounts.
- `generate_payloads()` can turn calculated swap amounts into low-level swap call payloads for an executor contract path, but the actual executor contract and flash-loan transaction orchestration are outside this repo's current completed surface.

## How A Future Flash-Loan Bot Would Fit

A production bot should be added as a thin orchestration layer around this package rather than buried inside pool math modules.

- Strategy scanner: choose token pairs, Curve pools, and external pools to evaluate.
- State refresh loop: keep candidate pools current from RPC, database snapshots, or event subscriptions.
- Profit calculator: use `UniswapCurveCycle` or a new Curve-focused helper to find candidate route size and profit.
- Simulation layer: test each candidate against a local fork or call-static style execution, including gas and flash-loan premium.
- Executor contracts: Solidity contracts for flash-loan borrow, swaps, repayment, and profit transfer.
- Transaction sender: private relay or protected RPC submission to reduce MEV risk.
- Risk controls: minimum net profit, max gas, stale-state checks, revert handling, per-route limits, and kill switch.
- VPS service: systemd or container process, logs, metrics, alerts, restart policy, and secret injection.

## Commands

- Install/sync dependencies: `uv sync`
- Run CLI help: `uv run degenbot --help`
- Run Python tests: `just test-python`
- Run Rust tests: `just test-rust`
- Run all tests: `just test-all`
- Run lint/type checks: `just lint`
- Format code: `just format`

Notes from upstream `AGENTS.md`:

- Use Red/Green TDD for refactors and features.
- Rust extension rebuilds automatically on import through maturin.
- Do not manually recreate the virtual environment after Rust changes.
- Use `from degenbot.logging import logger` for logging.
- Exceptions should inherit from `DegenbotError`.

## VPS Deployment Notes

This repo is not ready to run as an unattended money-moving VPS process yet. Before deployment, add:

- `.env.example` with non-secret variable names only.
- A runner entry point, for example `src/project_degen_bot/runner.py` or `scripts/run_strategy.py`.
- A config schema for RPC URLs, route lists, contract addresses, chain IDs, minimum profit, and gas limits.
- A contract package or external contract repo for flash-loan execution.
- Mainnet-fork tests for the exact Curve routes being targeted.
- A service file or container setup for the VPS.
- Structured logs and alerting around opportunity detection, simulation result, transaction hash, revert reason, and net PnL.

Never commit private keys, seed phrases, funded wallet files, production RPC credentials, or VPS secrets.
