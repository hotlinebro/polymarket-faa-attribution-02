# Polymarket FAA attribution forecast 02

Forecast Polymarket's FAA outage attribution Yes prices, 4 hours ahead,
as five quantiles (5%, 25%, 50%, 75%, 95%) per minute.

## Method

Price-only, no news, no network, fully deterministic (Python 3.13,
standard library only). `main.py` reads one case from
`/input/<id>.in` (or `$MAKEFUN_INPUT_DIR`) and writes
`/output/<id>.out` (or `$MAKEFUN_OUTPUT_DIR`).

- **Median**: the market's last price, pulled toward its
  board-sum-normalised share — but only when the board looks
  mutually exclusive (|board sum - 1| < 0.30). The pull is asymmetric
  (0.38x when the board is under-priced, 1.6x when over-priced), grows
  with horizon as (1 - exp(-h/110)), and is scaled by the volatility
  regime. The gate matters: the final's FAA markets are four
  independent questions whose Yes prices sum to ~0.10, not ~1, so an
  ungated pull toward 1 would be a large wrong upward bias.
- **Give-back**: after a 1-minute rise of >= 0.02 the price gives back
  the whole rise (time constant 90 minutes); falls of the same size
  persist.
- **Deadband**: median displacements under 0.0006 snap to the last
  price, with the spread compressed x0.8.
- **Spread**: a 0.65/0.35 blend of a fast (15-min) and slow (4-hour)
  EWMA-RMS of winsorised 1-minute changes, with a price-level floor
  and a 0.008 cap, then halved.
- **Fan**: Laplace quantiles, outer pair widened x1.1, inner pair
  narrowed x0.7, spread growing like sqrt(h).

## Verification

Local scorer on the six practice cases (NVIDIA event, the same cut as
the contest's practice): mean pinball loss **0.240411** (median error
0.6583, 90% coverage 0.8932), against the house baseline's 0.363589.
Holdout p04-p05: 0.1772 and 0.1019. Outputs are valid (all markets,
240 rows x 5 quantiles, in [0,1], non-decreasing, <= 1 MiB) and
byte-identical across two runs.
