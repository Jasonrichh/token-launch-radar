# Token Radar — make.fun bounty entry (token-radar-01)

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/Jasonrichh/token-launch-radar/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/Jasonrichh/token-launch-radar/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

A dashboard that finds pre-token protocols in a sealed DefiLlama snapshot, ranks them by
implied value, and shows how to farm each one. Built for the make.fun bounty
"Build the best token launch radar" (deadline Mon 14 Dec 2026 18:00, $1,325 in prizes).

## The model (radar-2)

`implied = round(TVL x category multiple x growth multiplier x maturity multiplier)`

- **Category multiple** — the median market-cap / TVL of the live-token protocols in the
  same category (at least 3 samples, else the global median). The market itself prices each
  category; no protocol name is ever special-cased.
- **Growth multiplier** — `1 + 0.5 x tanh(clamp(change_7d, -80, 400) / 50)`. A 7-day surge
  lifts the value, a slide cuts it, and tanh keeps a +4,448% outlier from counting more than
  a strong +120% run.
- **Maturity** — `0.75 + 0.25 x min(age_days, 1460) / 1460` (unknown age: 365 days). A protocol
  that has run a long time with no token is overdue for one.

The board covers the whole 40-protocol watchlist plus every other rankable pre-token
protocol below the floor (226 rows on this snapshot), with an 8-step farm plan per row:
wallet setup, the category's main action, a second qualifying action, weekly cadence, a
points-page check, the hold, claim safety, and a farm log with an exit plan. Each step
states its cost and its time.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json|directory>
```

Node 22.18+ (type stripping, standard library only). The program reads the snapshot,
writes `board.json`, `data.js`, copies `dashboard/` and `screenshots/`, and writes
`dossier.json` with the vendored make.fun library. Two runs produce byte-identical output.

## Layout

- `main.ts` — entrypoint (Node 22, stdlib only)
- `lib/model.ts` — the radar-2 model, the board builder and the farm plans
- `vendor/makefun-board.ts` — the kit's dossier library, byte for byte
- `dashboard/` — the static page (index.html, app.css, app.js; no network, no framework)
- `screenshots/` — the board, a protocol's detail view, and a filtered view
- `.makefun/owner.json` — the claim file; the wallet address must be filled in from the
  entrant's linked Solana wallet before submitting

## Submit (do not run without the entrant)

```sh
node <kit>/cli/makefun.mjs check --kit <kit> --repo <your repository> --transcripts transcripts.tar.gz
node <kit>/cli/makefun.mjs submit --competition token-radar-01 --repo-url https://github.com/<owner>/<repo> \
  --commit <sha> --title "Token Radar" --bundle transcripts.tar.gz --keypair wallet.json
```

Submission signs the entry message `make.fun entry v1` with the entrant's linked Solana
wallet and needs the transcripts of the agent sessions that built the entry.
