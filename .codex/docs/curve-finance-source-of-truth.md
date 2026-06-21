# Curve Finance Source Of Truth

Verified against the official Curve docs on 2026-06-21.

## Purpose

The official Curve documentation is the protocol source of truth for this fork's Curve arbitrage work:

https://docs.curve.finance/

Use this document as the local navigation layer for what we are trying to build in `project-degen-bot`. It does not replace Curve docs or contract source. It records which official sections matter, how they map into this repo, and what must be proven before any VPS-running arbitrage service is allowed to move funds.

## Source Priority

When Curve behavior is unclear or conflicts with local assumptions, use this order:

1. Official Curve docs and linked Curve contract source.
2. Live deployed contract behavior on the target chain.
3. Mainnet-fork tests against the exact pool and route.
4. This repo's existing implementation.
5. Notes, examples, old scripts, and informal assumptions.

The repo should adapt to Curve. Curve docs should not be bent to match the repo.

## Canonical Curve Docs

- Curve docs root: https://docs.curve.finance/
- Developer overview: https://docs.curve.finance/developer/documentation-overview
- Curve AMM overview: https://docs.curve.finance/developer/amm/curve-amm-overview
- Stableswap-NG overview: https://docs.curve.finance/developer/amm/stableswap-ng/overview
- Integrating Stableswap-NG: https://docs.curve.finance/developer/integration/stableswap-ng
- Curve Router NG: https://docs.curve.finance/developer/amm/router/curve-router-ng
- Integration overview: https://docs.curve.finance/developer/integration/overview
- AddressProvider: https://docs.curve.finance/developer/integration/address-provider
- MetaRegistry: https://docs.curve.finance/developer/integration/meta-registry
- Contract deployments: https://docs.curve.finance/developer/deployments
- Curve API docs: https://docs.curve.finance/developer/integration/api/curve-api

## What The Curve Docs Say That Matters Here

- Curve is an AMM/DEX for efficient trading of stablecoins and volatile assets across Ethereum and EVM chains.
- Curve AMM work is centered on StableSwap for near-parity assets and CryptoSwap for volatile pairs.
- Current AMM contracts include Stableswap-NG, Twocrypto-NG, Tricrypto-NG, pool factories, and routers.
- Stableswap-NG supports 2-8 coin pools, plain pools, metapools, standard ERC-20 tokens, rate-oracle tokens, rebasing tokens, and ERC-4626 vault tokens.
- Stableswap-NG adds price and D oracles, dynamic fees, `exchange_received`, and `get_dx`.
- Pool discovery can happen through factories for a specific pool family or through MetaRegistry for a cross-factory view.
- AddressProvider is the entry point for discovering important Curve registry contracts on supported chains.
- MetaRegistry aggregates Curve pool registries and exposes pool metadata such as coin lists and balances.
- For quoting, pools and views contracts expose `get_dy`, `get_dx`, `get_dy_underlying`, and `get_dx_underlying`.
- For execution, core Stableswap-NG paths include `exchange`, `exchange_received`, and metapool `exchange_underlying`.
- Curve Router NG can swap across up to five tokens in one transaction and exposes quote helpers.
- The deployments page is the official address directory, but it warns that some entries may be stale or incorrect, so live-chain verification is required.

## Build Target For This Fork

The immediate target is not "a generic trading bot." The target is a Curve-first arbitrage research and execution stack that can eventually support flash-loan-backed routes.

Build toward these components:

- Curve source registry: discover pools, pool type, coins, balances, decimals, asset types, implementations, and registry source.
- Quote layer: compare local Python math against official on-chain quote functions for the same pool state.
- Route scanner: evaluate candidate Curve routes and cross-DEX routes that include Curve.
- Profit model: subtract gas, flash-loan premium, slippage buffer, RPC latency risk, and failed-transaction risk.
- Simulation layer: run every candidate against a local fork or call-static style executor before submission.
- Executor contract: borrow, swap, repay, and return profit atomically.
- Transaction sender: use protected/private submission where needed to reduce MEV exposure.
- VPS runner: run only after the quote, simulation, executor, and monitoring layers exist.

## Repo Mapping

- `src/degenbot/curve/curve_stableswap_liquidity_pool.py` is the existing Curve pool model. It currently focuses on legacy Curve V1 StableSwap style behavior.
- `src/degenbot/arbitrage/uniswap_curve_cycle.py` is the existing mixed Uniswap/Curve arbitrage helper.
- `src/degenbot/arbitrage/types.py` defines the calculation result and swap amount value objects.
- `docs/arbitrage/` contains upstream arbitrage design notes.
- `docs/aave/flows/flash_loan*.md` contains Aave flash-loan protocol notes, but not a complete executor.
- New Curve-NG work should live in focused modules instead of overloading legacy classes without tests.

## Implementation Rules

- Do not hardcode Curve pool behavior from memory. Link the relevant official doc section in the issue, PR, or Codex note.
- Treat pool type as data: plain pool, metapool, StableSwap, CryptoSwap, NG, and legacy pools can differ.
- Treat dynamic fees as part of the quote. A static fee shortcut is acceptable only in tests that state the assumption.
- Treat metapool underlying indices carefully. `exchange_underlying` index mapping is not the same as direct `coins(i)` indexing.
- Treat rebasing, rate-oracle, and ERC-4626 assets as special cases until tested against live contract calls.
- Use official deployments and registries for discovery, then verify addresses on-chain.
- Every Curve quote implementation must have tests comparing local output to contract `get_dy` or `get_dx` for representative pools.
- Every executor path must be tested on a fork before any mainnet transaction path is enabled.
- Never add private keys, funded-wallet files, RPC secrets, or Hetzner/API credentials to this repo.

## Suggested Build Phases

### Phase 1: Source Mapping

- Pick one target chain, likely Ethereum mainnet first.
- Pick initial pools and tokens deliberately, not from vague opportunity assumptions.
- Build a local address registry from official deployments, AddressProvider, MetaRegistry, and live calls.
- Record exact pool type and ABI surface for each target pool.

### Phase 2: Curve Quote Correctness

- Add tests that call official pool/view quote functions.
- Compare those quotes with local `CurveStableswapPool` calculations.
- Identify gaps between legacy Curve V1 handling and Stableswap-NG requirements.
- Add new modules where the NG model diverges from the legacy implementation.

### Phase 3: Route And Profit Scanner

- Scan only approved route shapes at first.
- Include gas estimates, premium estimates, slippage buffer, and stale-state checks.
- Emit structured candidate records even when no trade is profitable.

### Phase 4: Fork Simulation

- Build an executable simulation for each candidate route.
- Fail closed when local quote, on-chain quote, and fork execution disagree.
- Track gross profit, gas, premium, net profit, and revert reason.

### Phase 5: Flash-Loan Execution

- Add or reference a Solidity executor contract.
- Keep flash-loan lender concerns separate from Curve route math.
- Require fork tests for borrow, swaps, repayment, and profit withdrawal.

### Phase 6: VPS Runner

- Add a minimal service runner only after the previous phases pass.
- Run from `/opt/project-degen-bot` on the current VPS.
- Use environment variables or a secret manager for credentials.
- Log opportunity detection, simulation, submission, result, and net PnL.

## Open Decisions

- Which chain is first: Ethereum mainnet, Base, Arbitrum, or another Curve-supported chain?
- Which flash-loan lender is first: Aave V3, Balancer, Uniswap V3 flash swap, or another source?
- Which Curve pool family is first: legacy StableSwap, Stableswap-NG, Twocrypto-NG, or Tricrypto-NG?
- Are we using Curve Router NG for execution, direct pool calls, or both?
- What is the minimum net profit per transaction after all costs?

## Definition Of Ready For Live Money

This repo is not ready for live money until all of the following are true:

- Official Curve docs and contract source are linked for each supported route type.
- Local quotes are tested against on-chain quote functions.
- Fork execution matches expected output for the full flash-loan path.
- Net profit calculation includes gas and flash-loan premium.
- Private transaction submission is decided and tested.
- VPS runner has logs, restart policy, alerts, and a kill switch.
- Secrets are injected outside Git.
- A dry-run mode has operated without submitting transactions.
