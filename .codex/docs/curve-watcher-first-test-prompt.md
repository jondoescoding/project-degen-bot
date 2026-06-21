# CURVE-SETUP First Test Prompt

Use this prompt to start the next development thread for the first Curve watcher prototype.

```text
Start on the Linux VPS checkout, not the Windows working copy.

SSH:
ssh -i ~/.ssh/promptbank_hetzner_face_swap_ed25519_v2 root@178.105.108.228
cd /opt/project-degen-bot

Use Beads for task state:
bd prime
bd show degen-ldu.1 --long
bd update degen-ldu.1 --claim

Objective:
Implement CURVE-SETUP first test: a dry-run two-token quote watcher based on this repo's current degenbot/Web3 stack. It should replicate the useful behavior of jondoescoding/genericTokenWatcher by repeatedly quoting a configured token pair/path and reporting whether an arbitrage threshold may exist.

Important constraints:
- Treat https://github.com/jondoescoding/genericTokenWatcher as a behavior reference only.
- Do not port Brownie or the old genericTokenWatcher dependency model.
- Use current repo primitives first: degenbot.connection, Erc20Token, CurveStableswapPool, and direct Web3 calls only where this repo has no wrapper yet.
- Keep it dry-run and non-custodial: no private keys, approvals, transactions, flash loans, or trading.
- Use .codex/docs/curve-finance-source-of-truth.md and official Curve docs as the protocol source of truth.
- Accept chain/RPC config, token_in, token_out, amount_in, interval, threshold, and quote source/path from CLI or config.
- Print structured quote data and threshold status to stdout; no Telegram dependency for the first pass.
- Place the watcher where it best fits this repo, likely scripts/curve_quote_watcher.py or a small module plus script.
- Include a .codex note or README example command.

Verification:
- Add focused unit tests for pure config parsing and threshold logic.
- Run the watcher on the VPS in dry-run mode when RPC and Curve pool/router inputs are available.
- If live inputs are missing, stop and ask for chain, RPC URL, token addresses, amount, threshold, and Curve pool/router address.

Current Beads acceptance criteria:
Runs from /opt/project-degen-bot on the VPS; uses the current degenbot/Web3 stack, not Brownie; treats genericTokenWatcher as a behavior reference only; accepts chain/RPC config, token_in, token_out, amount_in, interval, threshold, and quote source/path from CLI or config; uses degenbot.connection, Erc20Token, CurveStableswapPool, or direct Web3 calls only where the repo has no wrapper; prints structured quote and threshold status; requires no private key, approvals, or transaction sending; includes a .codex note or README example command; is verified on Linux/VPS with unit tests for pure logic and a dry-run smoke when RPC/pool config is available.
```

