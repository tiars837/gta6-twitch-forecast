# GTA VI Twitch Forecast

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/tiars837/gta6-twitch-forecast/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/tiars837/gta6-twitch-forecast/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

Forecasts Grand Theft Auto VI's launch-day Twitch viewers hour by hour —
for each of hours 2 to 24 (UTC) of 19 November 2026, five quantiles
(5%, 25%, 50%, 75%, 95%) of that hour's concurrent viewers — built for the
[make.fun bounty "Forecast GTA VI's launch-day Twitch viewers hour by hour"](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
($250).

## How it works

A launch day's *shape* is mostly the same from one big game to the next:
viewers unlock country by country at midnight local time, so the UTC day
ramps as regions come online, peaks, then fades overnight. The *level*
differs wildly, but each hour's viewers as a ratio to hour 1's is stable
within launches of a similar size. For every hour h = 2..24 the program:

1. takes every comparable past launch in the case, reads its release-day
   hour-1 print and its hour-h print, and forms the log ratio
   `L = ln(viewers_h / viewers_1)`;
2. weights each launch by how close its hour-1 print is to this case's
   (a Gaussian kernel in log scale, sigma 0.9), then takes the weighted
   mean and spread of the log ratios at that hour;
3. anchors them on this case's own hour-1 print:
   `q_tau(h) = p1 * exp(mu_h + z_tau * sd_h)`, with the spread shrunk
   slightly (×0.9).

When too few comparables carry weight, it falls back to the house
baseline's widening spread around the median ratio, so every hour still
gets five valid quantiles. No network, no model call, fully deterministic:
the same case always writes the same bytes.

## Local test (leave-one-out over the kit's comparables)

The kit's practice answers stay on the server, so the model was validated
by leave-one-out over its 38 comparables: each launch in turn was treated
as the target, forecast from the other 37, and scored with the contest's
pinball loss on the log scale (148 pseudo-cases, 17,020 items):

| | loss |
|---|---|
| This forecaster | **0.394** |
| House baseline (flat at hour-1 print) | 0.930 |
| Improvement | **57.6%** |

On launches as big as GTA VI's hour 1 is expected to be (hour-1 print
≥ 50,000): this forecaster 0.204 vs baseline 0.251. 90% coverage 0.871
(honest target ≈ 0.9). Runtime ≈ 0.13 s per case (limit 8 s).

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out node main.ts
```

Node 22, standard library only. Reads every `*.in` in the input directory
and writes `<id>.out` (`{"hours": [{"hour": h, "q": [q05, q25, q50, q75, q95]}, ...]}`)
for hours 2..24.

## Layout

- `main.ts` — the whole forecaster (entrypoint).
- `README.md`, `LICENSE` — this file, MIT.

## License

MIT — see [LICENSE](LICENSE).
