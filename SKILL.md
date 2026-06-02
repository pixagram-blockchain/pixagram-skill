---
name: pixagram
description: Use when interacting with the Pixagram blockchain — a Hive fork. Triggers on tasks that mention Pixagram, PIXA / PXS / VESTS tokens, the api.pixagram.com or pixagram.dev endpoints, hived/Hivemind on this fork, or any account/post/witness operation on this chain. Tells the agent which endpoint to hit, which token names to use, which fields/accounts are renamed, the genesis allocations and reward-curve tuning that differ from upstream Hive, and that the standard Hive API surface applies for everything else.
---

# Pixagram

A fork of [Hive](https://hive.io). The standard Hive RPC surface (`condenser_api.*`, `database_api.*`, `bridge.*`, `follow_api.*`, `tags_api.*`) works — only the renames and tuning below differ.

## Endpoints

| Network | RPC |
|---|---|
| Mainnet | https://api.pixagram.com |
| Testnet | https://pixagram.dev |

Both serve full Hive-compatible JSON-RPC. `bridge.*`, `follow_api.*`, `tags_api.*` and select `condenser_api.*` social methods are routed to a **Hivemind** indexer behind the same endpoint.

## Differences from Hive

### Tokens & addresses
| | |
|---|---|
| Liquid native | **PIXA** (replaces HIVE) |
| Stable | **PXS** (replaces HBD) |
| Staked | **VESTS** (same as Hive) |
| Public-key prefix | **`PIX`** (e.g. `PIX6LLegb…`) |
| Chain ID | derived from ASCII string `"pixagram"` (padded to 32 bytes) |

When building legacy-format asset payloads, send `PIXA` / `PXS` as the on-wire symbol bytes (not `STEEM` / `SBD` that older Hive clients hardcode). HF26 NAI-format assets work as-is.

### API field renames
Jussi rewrites these in responses (and accepts the new names in requests):

| Hive | Pixagram |
|---|---|
| `hbd_balance` | `pxs_balance` |
| `hbd_exchange_rate` | `pxs_exchange_rate` |
| `hbd_interest_rate` | `pxs_interest_rate` |
| `reward_hive` / `reward_hbd` | `reward_pixa` / `reward_pxs` |
| `total_vesting_fund_hive` | `total_vesting_fund_pixa` |
| `current_hbd_supply` | `current_pxs_supply` |
| `dhf_interval_ledger` | `dpf_interval_ledger` |

Symbols inside response strings (`"1.000 HBD"` etc.) are also normalized to PIXA/PXS.

### Governance
- **DPF** (Decentralized Proposal Fund) replaces Hive's DHF. Same mechanics — funded by `proposal_fund_percent = 15%` of per-block inflation.
- Treasury account is **`pixa.omnibus`** (replaces `steem.dao` / `hive.fund`). Holds PXS only; spendable only via approved DPF proposals.

### Reward curves (tuned)
| | Value | Note |
|---|---|---|
| `author_reward_curve` | `convergent_linear` | same as Hive post-HF21 |
| `curation_reward_curve` | `convergent_square_root` | same as Hive post-HF21 |
| `content_constant` (`s`) | **`2500`** | **drastically smaller** than upstream's `2_000_000_000_000` — tuned for pre-mainnet rshare scale so small accounts produce non-zero rshares |
| `percent_curation_rewards` | `40%` | of the content reward pool |

### Communities (Hivemind)
Community names must match **`portal-[123]\d{4,6}`** (e.g. `portal-100001`) — Pixagram **kept upstream's `portal-` prefix**, did NOT rename to `pixagram-`. Fresh chain → no communities exist until `community_create` ops are broadcast.

## Init / system accounts & genesis allocations

| Account | Genesis allocation | Notes |
|---|---|---|
| `initminer` | 0 PIXA, 0 PXS, 0 VESTS at block 0 — accumulates VESTS via producer rewards | Sole witness on pre-mainnet |
| `pixa.ico` | **50,000,000 VESTS** | ICO / sales pool |
| `pixa.team` | **25,000,000 VESTS** | Team vested allocation |
| `pixa.omnibus` (treasury) | **~250,000 PXS** | DPF treasury — value-equivalent of 25M PIXA at the genesis median feed (1 PXS = 102 PIXA) |
| `miners`, `null`, `temp`, `steem` | none | Hive-inherited system placeholders; empty key_auths → unspendable. `steem` exists only as a legacy recovery_account fallback target. |

### Initial supply summary (block 0)
| | |
|---|---|
| Liquid PIXA | **0** — no liquid PIXA at genesis; everything flows from inflation / powerdowns |
| Vested PIXA (in `total_vesting_fund_pixa`) | **75,000,000 PIXA** backing 75M VESTS |
| Total VESTS | **75,000,000** (50M `pixa.ico` + 25M `pixa.team`) |
| Liquid PXS | **~250,000** (in `pixa.omnibus`) |
| **VESTS : PIXA at genesis** | **1 : 1** in display units (= 1,000,000 raw microVESTS per display PIXA). Macro: `HIVE_INITIAL_VESTING_PRICE = price(VEST_asset(1000), HIVE_asset(1))`. Diverges from Hive/Steem's historical `1 STEEM ≈ 0.0018 VESTS` genesis ratio. |
| **Genesis median feed** | `1 PXS = 102 PIXA` seeded in source so conversions work before witnesses publish |

The vesting ratio drifts upward over time as inflation flows into `total_vesting_fund_pixa` without issuing new VESTS — the standard Hive staking-yield mechanism.

## Quick examples

```bash
# Global chain state
curl -s -X POST https://api.pixagram.com -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"condenser_api.get_dynamic_global_properties","params":[],"id":1}'

# Account (note: pxs_balance not hbd_balance)
curl -s -X POST https://api.pixagram.com -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"condenser_api.get_accounts","params":[["initminer"]],"id":1}'

# Treasury balance (DPF)
curl -s -X POST https://api.pixagram.com -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"condenser_api.get_accounts","params":[["pixa.omnibus"]],"id":1}'

# Reward fund (curves, constants, balance)
curl -s -X POST https://api.pixagram.com -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"condenser_api.get_reward_fund","params":["post"],"id":1}'

# Median price feed
curl -s -X POST https://api.pixagram.com -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"condenser_api.get_current_median_history_price","params":[],"id":1}'

# Social feed (Hivemind)
curl -s -X POST https://api.pixagram.com -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","method":"bridge.get_ranked_posts","params":{"sort":"trending","tag":"","limit":20},"id":1}'
```

For anything else, refer to [Hive developer docs](https://developers.hive.io/) — methods and ops are identical.
