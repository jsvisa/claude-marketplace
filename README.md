# jsvisa Claude Marketplace

Claude Code plugin marketplace by [jsvisa](https://github.com/jsvisa).

## Add this marketplace

```bash
/plugin marketplace add jsvisa/claude-marketplace
```

## Available plugins

### evm-decoder

Decode EVM transaction calldata, event logs, and fetch verified contract ABIs across any EVM chain.

```bash
/plugin install evm-decoder@jsvisa-marketplace
```

**What you get:** Ask Claude to decode a transaction, an event log, or fetch a contract ABI — it calls [eth-decoder.vercel.app](https://eth-decoder.vercel.app) automatically.

→ [jsvisa/evm-decoder-skill](https://github.com/jsvisa/evm-decoder-skill)

---

### solexplain

Explain Solana transactions in human-readable form — programs involved, SOL/token transfers, and protocol-specific decoded actions.

```bash
/plugin install solexplain@jsvisa-marketplace
```

**What you get:** Ask Claude to explain a Solana transaction signature — it uses `solexplain` to decode programs, transfers, and protocol details (Jupiter, Raydium, Orca, Drift, Pump.fun, Wormhole, and more).

→ [jsvisa/solexplain](https://github.com/jsvisa/solexplain)

---

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
