# RSGP-USDC pool depth — auto-refreshed to Dune

Reproduces Dune query 7685210 (hourly / N-minute pool depth) straight from a Base RPC and
uploads the CSV to Dune every day via GitHub Actions.

- `main.go` — the pipeline (see `CLAUDE.md` for flags and semantics)
- `state.go` — incremental mode: checkpoint + mint/burn snapshot so each run only fetches new blocks
- `upload_dune.py` — pushes a CSV to Dune's table upload API (replaces the table)
- `.github/workflows/refresh.yml` — daily 02:30 UTC run for RSGP-USDC (0x8e859b6cf564c4ac2b848693c85c496e81dbf6c8, USDC = token1) at 15-minute buckets
- `state/` — incremental checkpoint, created by the first run (delete it to force a full rescan)

## Pool

| | |
|---|---|
| Pool | `0x8e859b6cf564c4ac2b848693c85c496e81dbf6c8` (Aerodrome Slipstream, post-2026-10-21 factory `0x9cb220581b91b0c83223012fe69688b92ad07a76`) |
| Tokens | RSGP `0x44b63af715d5ef02cc3646a6ce96875e11d69e85` (token0, 18 dec) / USDC (token1, 6 dec) |
| Tick spacing / fee | 100 / 30000 |
| Deployed | 2026-10-27 13:37:47 UTC (block 53,159,460) |
| First swap | 2026-10-29 14:45 UTC |

Data is fetched from `mainnet.base.org` with `-filter-logs` (that endpoint refuses `eth_getLogs`); see
`CLAUDE.md`. Verified locally on 2026-10-29: the liquidity the script rebuilds from Mint/Burn matches
the pool's on-chain `liquidity()`.

## Setup

1. Repo → Settings → Secrets and variables → Actions → **New repository secret**:
   `DUNE_API_KEY` = your Dune API key (Dune → Settings → API).
2. Actions tab → "Refresh RSGP-USDC depth" → **Run workflow** (or wait for the daily schedule).
3. Query it on Dune:

```sql
SELECT date_parse(hour, '%Y-%m-%dT%H:%i')   -- Dune stores hour as text '2026-10-29T14:45' AS ts, *
FROM dune.forcemulti.dataset_rsgp_usdc_depth_15m
ORDER BY 1
```
