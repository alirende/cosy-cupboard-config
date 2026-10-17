# Cosy Cupboard remote config

The live tuning for Cosy Cupboard: ad kill switches, caps and pacing, booster prices,
coin rewards, undo allowances and the revive. The game fetches it over HTTPS at launch:

    https://alirende.github.io/cosy-cupboard-config/v1/config.json

**This file is production.** Players pick up a change at their next launch, or at once
if the game is open and fetching. Allow about ten minutes from push to players: Pages
rebuilds in a minute or two and its CDN caches the file for ten (`max-age=600`), so a
kill switch is minutes, not seconds. Changes to caps, pacing or prices are reviewed
before they are published. Never put secrets, ad unit ids or anything about players in this repo:
it is public.

## How the game reads it

- The bundled values in the game are the fallback. If a fetch fails (offline, 404, not
  JSON, a different `schema`), the game keeps what it had: the last good fetch from its
  cache, or the bundled values.
- Each key is checked on its own. A value of the wrong type, a fraction where a count
  belongs, or a value outside its range keeps the bundled default, and the rest still
  apply. Unknown keys are ignored, so a newer key does no harm to older builds.
- A key left out keeps its bundled default.
- `version` identifies the published document. Bump it on every publish.
- A breaking change to the format bumps `schema` and moves to `v2/config.json`; builds
  that read schema 1 keep reading `v1/`.

## Kill switches

- `ads_enabled: false` turns off every ad placement at once.
- `ad_<placement>_enabled: false` turns off one placement (`extra_moves`, `undo`,
  `life_refill`, `double_coins`, `interstitial`). The game stops offering it.
- `lives_enabled: false` (the default) turns lives off: no board costs a life or waits
  for one, and Home shows no lives. `true` brings them back as each player left them.
- `rooms_enabled: false` stops selling and offering rooms; owned rooms stay.
- `crash_reports_enabled: false` stops crash reports even for players who opted in.
- `booster_extra_shelf_enabled: false` removes the extra-shelf offer from the stuck
  drawer.

## Keys

The table matches the game's built-in definitions. `v1/config.json` still carries
`coin_per_spare_move`, `perfect_bonus` and `win_base` from the old reward model; the game
ignores unknown keys, so they do nothing (removing them is a config change).

| Key | Default | Range | Note |
| --- | --- | --- | --- |
| `ads_enabled` | false | on / off | Off until real ads are in the game |
| `ad_extra_moves_enabled` | true | on / off |  |
| `ad_extra_moves_daily_cap` | 10 | 0 to 100 |  |
| `ad_undo_enabled` | true | on / off |  |
| `ad_undo_daily_cap` | 30 | 0 to 200 |  |
| `ad_life_refill_enabled` | true | on / off |  |
| `ad_life_refill_daily_cap` | 5 | 0 to 50 |  |
| `ad_double_coins_enabled` | true | on / off |  |
| `ad_double_coins_daily_cap` | 20 | 0 to 100 |  |
| `ad_interstitial_enabled` | true | on / off |  |
| `interstitial_start_level` | 6 | 1 to 1000 |  |
| `interstitial_every_levels` | 3 | 1 to 100 |  |
| `interstitial_cooldown_sec` | 75 | 30 to 3600 |  |
| `revive_moves` | 5 | 1 to 30 | added by the extra-moves ad |
| `booster_extra_shelf_enabled` | true | on / off | Offer the extra shelf in the stuck drawer |
| `booster_extra_shelf_cost` | 100 | 1 to 10000 | Price of the extra shelf in coins |
| `booster_extra_shelf_per_level` | 1 | 1 to 5 | Times it can be bought per board |
| `lives_enabled` | false | on / off | Off: no board costs a life or waits for one, and Home shows no lives |
| `rooms_enabled` | true | on / off | Off: no room is sold or offered; owned rooms stay |
| `rooms_cost_coins` | true | on / off |  |
| `room_bathroom_cost` | 500 | 1 to 100000 |  |
| `room_study_cost` | 600 | 1 to 100000 |  |
| `room_bedroom_cost` | 850 | 1 to 100000 |  |
| `room_living_cost` | 1100 | 1 to 100000 |  |
| `coins_tidy` | 10 | 0 to 1000 | Coins for a win with one mark |
| `coins_close` | 12 | 0 to 1000 | Coins for a win with two marks |
| `coins_perfect` | 16 | 0 to 1000 | Coins for a win with three marks (the optimal move count) |
| `rating_close_pct` | 10 | 0 to 100 | How far over the optimal count still earns two marks, in percent (at least one move) |
| `rating_close_pct_hidden` | 50 | 0 to 200 | The same for boards with hidden cups |
| `daily_base` | 20 | 0 to 1000 | Coins for the daily challenge |
| `daily_streak_bonus` | 5 | 0 to 1000 | Extra coins per day of streak |
| `daily_streak_cap` | 7 | 0 to 365 | Days of streak that count toward the bonus |
| `free_undos` | 5 | 0 to 100 | per attempt |
| `crash_reports_enabled` | true | on / off | Off: crash reports stop, even for players who turned them on |
| `limit_free_levels` | 5 | 0 to 100 | The first levels have no move limit; 0 gives every level one |
| `ad_undos` | 5 | 1 to 100 | added by one rewarded undo ad |

The ranges are guard rails enforced by the game, not suggestions: for example the
interstitial cooldown cannot go under 30 seconds and nothing can cost zero coins.

## Publishing a change

1. Edit `v1/config.json` and bump `version`.
2. Check it is valid JSON: `python3 -m json.tool v1/config.json`.
3. Commit and push to `main`. Players see it within about ten minutes (see above).
