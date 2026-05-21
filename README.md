# jsvisa Claude Marketplace

Claude Code plugin marketplace by [jsvisa](https://github.com/jsvisa).

## Add this marketplace

```bash
/plugin marketplace add jsvisa/claude-marketplace
```

## Available plugins

<!-- plugins-start -->
### evm-decoder `v1.1.4`

Decode EVM transaction calldata, event logs, and fetch verified contract ABIs across any EVM chain via eth-decoder.vercel.app

```bash
/plugin install evm-decoder@jsvisa-marketplace
```

→ [jsvisa/evm-decoder-skill](https://github.com/jsvisa/evm-decoder-skill)

---

### solexplain `v0.2.3`

Explain Solana transactions in human-readable form — programs, SOL/token transfers, and protocol-specific actions (Jupiter, Raydium, Orca, Drift, Pump.fun, and more)

```bash
/plugin install solexplain@jsvisa-marketplace
```

→ [jsvisa/solexplain](https://github.com/jsvisa/solexplain)

---

### debank `v1.1.1`

Fetch a wallet's DeFi portfolio from DeBank — total balance, token holdings, and protocol positions across all chains. Supports CSV export (simple or full with token addresses, adapter IDs, controllers, and on-chain metadata via API interception)

```bash
/plugin install debank@jsvisa-marketplace
```

→ [jsvisa/debank-skill](https://github.com/jsvisa/debank-skill)

---

### etherscan `v1.0.0`

Query contract creation info, verified ABIs, source code, account balances, transaction history, token transfers, and block lookups on Ethereum and EVM chains via Etherscan v2 API. Uses curl and jq with multi-key rotation support.

```bash
/plugin install etherscan@jsvisa-marketplace
```

→ [jsvisa/etherscan-skill](https://github.com/jsvisa/etherscan-skill)

---
<!-- plugins-end -->

## Adding more plugins

To add a new plugin, create a repo with this structure:

```
your-skill/
  package.json          # { "name": "...", "version": "1.0.0", "type": "module" }
  skills/
    your-skill-name/
      SKILL.md          # frontmatter: name + description
  README.md
  LICENSE
```

Then add an entry to `.claude-plugin/marketplace.json` in this repo and open a PR.
