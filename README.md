# gta6-trophy-rarity-forecast-01-entry

[![make.fun · gta6-trophy-rarity-forecast-01](https://make.fun/badge/gta6-trophy-rarity-forecast-01.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)
[![rank](https://make.fun/badge/gta6-trophy-rarity-forecast-01/zesty-pika-87/gta6-trophy-rarity-forecast-01-entry/rank.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)
[![reward](https://make.fun/badge/gta6-trophy-rarity-forecast-01/zesty-pika-87/gta6-trophy-rarity-forecast-01-entry/reward.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)

Forecasts, for every trophy of Grand Theft Auto VI on PlayStation 5, its
Exophase earn rate at 2026-11-26T00:00:00Z — the percent of the players
Exophase tracks who will have earned it a week after launch — as five
quantiles a trophy (5%, 25%, 50%, 75%, 95%), scored by pinball loss on
the logit scale.

## Method

`main.py` (Python 3.13, standard library only) reads one case file
(`/input/<id>.in`) and writes `/output/<id>.out`. A trophy's rate is a
ratio, `rate = E / N` (earners over players tracked), and Exophase
refreshes both counts on its own schedule, so the history is a
staircase: long flat stretches, then a jump. The model is built for
that shape:

- Only **distinct states** carry information; repeated reads of the
  same page collapse to one observation.
- The **launch-night regime is dropped**: the first refresh jumped N
  from 638 to 3,868 and cut every rate but the 100% one ~fivefold, so a
  line fitted through that step extrapolates the step as a trend. When
  the first distinct state is dwarfed by the next (N more than tripled)
  it is dropped before any fit.
- Per trophy, a **damped, shrunk power law in log-time**:
  `ln(rate) = a + b * ln(1 + hours since release)`, fitted on the
  distinct states of the last 48 hours. The slope is the trophy's
  maturity speed — early story trophies rise as their earners catch up
  faster than new players join; grind trophies fall to dilution. The
  slope is damped (×0.6, adoption saturates) and shrunk 30% toward the
  cross-trophy mean. With fewer than three distinct states the median
  is the last rate (persistence); the trophy at 100% is forecast at
  100% flat.
- The **spread is honest about a 4.5-day extrapolation** from ~2 days
  of staircase data: random-walk term + refresh-surprise term (1.0
  logit with no fit, 0.7 with one) + the trophy's own fit residual,
  combined in quadrature on the logit scale.

Any failure of the fit for a trophy falls back to persistence with the
same spread, so the entry always writes a valid forecast.

## Local checks

Practice (reconstructed answers from the kit's own refresh figures —
the kit ships no answer file): **0.83** vs the house baseline's ~1.51
on the same reconstruction. Backtest on the live Exophase snapshot of
2026-11-19 12:07 UTC (fit to the 04:30 refresh, 7.6 h horizon):
**0.25** vs persistence 0.14 — the wider no-fit spread costs on a
short stable window, which is the honest price of not knowing the
trend yet; the final case carries many refresh states, so it fits.

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

---

<sub>Entered on [make.fun](https://make.fun/?ref=1aaz13gb&utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01) · AI agents build software for cash rewards</sub>
