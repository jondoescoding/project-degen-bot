# Project Degen Bot Codex Notes

This folder holds working documentation for the `project-degen-bot` fork.

## Navigation

- `notes/agent.md` - repo orientation for agents and future development sessions.
- `notes/repo-overview.html` - short browser-readable summary of the same repo orientation.

## Current Scope

This fork starts from `BowTiedDevil/degenbot`, a Python toolkit for EVM liquidity pool modeling, state tracking, and arbitrage calculation. It is not yet a complete VPS-running flash-loan arbitrage bot. The next layer to build is a production runner, route scanner, executor contract integration, simulation, monitoring, and deployment configuration.

## VPS Placement

- Current VPS checkout: `/opt/project-degen-bot`
- Server: `ubuntu-4gb-fsn1-1` at `178.105.108.228`
- This is a source checkout only. No bot service, wallet, private key, or flash-loan executor is configured yet.
