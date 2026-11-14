# Token Radar — token-radar-01

Every protocol DefiLlama tracks that still records no token, ranked by implied launch value.

## What it is

- **Board:** 213 rows — the full 40-protocol watchlist (TVL ≥ $50M, no token) plus every
  pre-token protocol below the floor that meets the brief's definition of a launch candidate
  (173 early rows). No top-N cut: the radar ranks the whole set it finds.
- **Model (radar-2):** `implied = TVL × base × growth × maturity × breadth`
  - `base` is **calibrated, not chosen**: the median market-cap/TVL ratio of the snapshot's own
    live-token protocols (0.2138 from 128 comparables in this snapshot; 0.25 fallback below 10).
    A negative market-cap entry in the snapshot is excluded by the `mcap > 0` rule.
  - `growth` = `1 + clamp(change_7d, −60, 120)/100 × 0.5`
  - `maturity` = `0.6 + min(age_days, 1095)/1095 × 0.4` (unknown age: 365 days)
  - `breadth` = `1 + min(chains − 1, 4) × 0.03` (more chains, wider reach)
  - No protocol name is special-cased; the same function scores every row.
- **Farm plans:** seven steps per protocol, derived from its category, chains, TVL and age —
  capital by TVL, gas priced by main chain, hold by age, with a second, different qualifying
  action per category.
- **Dashboard:** a static page (`dashboard/`) that renders the whole board with no network —
  board view, per-protocol detail view, filtered views (search, category, chain, watchlist-only),
  a live-token view, the model's formula, calibration and build funnel, and a category
  breakdown of combined implied value.

## How the board is built

The sealed snapshot holds 758 protocols. The funnel to the board:

| Step | Count |
| --- | --- |
| Protocols in the snapshot | 758 |
| Record no token (no market cap, no CoinGecko id, no symbol) | 342 |
| On the sealed watchlist (TVL ≥ $50M) | 40 |
| Pre-token above the floor but outside the watchlist (not ranked) | 92 |
| Pre-token below the floor | 210 |
| Removed below the floor by the brief's rule | 37 |
| **Rows on the board** | **213** |

The 37 removed below the floor are exchanges (15), versioned pools of another
protocol (9), products whose own name says they are a wrapped or staked derivative (12),
and one exchange-branded protocol. The watchlist itself is sealed and governs the
above-floor set; the early rule does not re-impose the watchlist's category screen,
because bridges and liquid-staking protocols do launch genuine governance tokens —
only products whose own name says they are a wrapped or staked derivative are removed.

The top of the board on this snapshot: Steakhouse Financial ($520.7M implied, $2.75B TVL),
Gauntlet ($310.1M), Veda ($298.2M), USDD ($194.2M), Upshift ($107.0M). The smallest row,
Monarq, carries $0.62M implied. Risk Curators hold the most combined implied value
($1.18B across 33 rows), ahead of Onchain Capital Allocators ($477.5M) and RWA ($241.3M).

## Farm plans

Each row carries seven steps, each an action with a cost and a time:

1. Fresh wallet, seed phrase offline (under $3 gas, 5 minutes).
2. The category's main action — supply, provide liquidity, bridge, stake, deposit margin,
   mint, buy cover — with capital by TVL ($50 / $100 / $150 / $250) and gas priced by the
   protocol's main chain (Ethereum ~$12, Solana ~$0.05, major L2s ~$1, others ~$0.50).
3. A second, different qualifying action for the same category (borrow against the supply,
   swap in the pool, claim and restake rewards, close and reopen the hedge).
4. One qualifying action a week.
5. Hold for 30–60 days by the protocol's age, so the activity spans more than one snapshot.
6. Check the protocol's own points page; claim only from the link it publishes, never pay
   to claim.
7. Keep a farm log: wallet, dates, every transaction.

The dashboard regenerates each plan from the row's own figures, and the detail view
recomputes the row's implied value in the page and confirms it matches `board.json`.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input /path/to/snapshot.json
```

Node 22, standard library only. Writes `board.json`, `data.js`, `dashboard/`, `screenshots/`
and `dossier.json` (last) to the output directory. The program refuses a board that misses
a watchlist slug, breaks rank order, or carries a farm step outside 20–600 characters.

## Check it

```sh
node <kit>/cli/makefun.mjs check --repo . --bundle transcripts.tar.gz
```

## Files

- `main.ts` — entrypoint: reads the snapshot, builds the board, writes the outputs.
- `lib/model.ts` — the radar-2 model, the board builder, the farm plans, the snapshot parser.
- `vendor/makefun-board.ts` — the kit's dossier builder (unchanged from the kit).
- `dashboard/` — the static page (index.html, app.css, app.js).
- `screenshots/` — board, detail and filtered views of the dashboard.
- `.makefun/owner.json` — claim file naming the agent wallet linked to the entrant's account.

## License

MIT — see `LICENSE`.
