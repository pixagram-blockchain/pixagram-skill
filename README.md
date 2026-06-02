# pixagram-skill

A Claude Code / Codex skill that gives an AI agent the context it needs to interact with the [Pixagram](https://pixagram.com) blockchain — a Hive fork.

The skill covers:
- Mainnet (`api.pixagram.com`) and testnet (`pixagram.dev`) RPC endpoints
- Token renames vs Hive (PIXA, PXS, VESTS, `PIX` pubkey prefix)
- API field renames Jussi performs (`pxs_balance`, `dpf_interval_ledger`, …)
- Genesis / system accounts (`initminer`, `pixa.ico`, `pixa.team`, `pixa.omnibus`, …)
- Hivemind routing for `bridge.*` / `follow_api.*` / `tags_api.*`
- Curl examples for the most common queries

The skill is intentionally brief — it points at [Hive's developer docs](https://developers.hive.io/) for everything that didn't change.

## Install

### Claude Code

```bash
git clone git@github.com:pixagram-blockchain/pixagram-skill.git ~/.claude/skills/pixagram
```

Restart Claude Code (or open a new conversation). The skill becomes available automatically and the agent will invoke it whenever a task mentions Pixagram, PIXA/PXS/VESTS, the public endpoints, or any account/post/witness operation on this chain.

### Codex / other agents

Point your agent at `SKILL.md` as a system / context document, or symlink it into wherever your agent loads skills from.

## Updating

```bash
cd ~/.claude/skills/pixagram
git pull
```

## Contributing

PRs welcome. Keep the skill **brief** — it should give the agent the deltas from Hive and the endpoints, not be a full reference. The principle: anything documented identically by Hive belongs in the Hive docs, not here.
