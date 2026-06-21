# AI Context

This file helps Claude (or any AI session) understand the repository well enough to edit it safely without re-deriving everything. It is **not** end-user documentation — see `README.md` and `INSTALLATION.md` for that.

## Repository purpose

Two Home Assistant dashboard tiles ("Pool Chemistry (main)" and "Pool Chemistry (supporting)") that visualise a Tuya 7-in-1 pool tester. Each water-chemistry reading is rendered as a horizontal gauge bar by a `custom:html-template-card`. The repo distributes the card YAML plus docs; it is **not** a HACS-installable plugin and ships **no** integration code.

**Companion project:** none.

## File structure

```
.
├── README.md                 Shop-window overview (benefits, features, how-it-works)
├── INSTALLATION.md           Step-by-step install (Claude path + manual) and troubleshooting
├── CHANGELOG.md              Keep-a-Changelog history; versions live here
├── card.yaml                 CANONICAL current card — both grid sections, 7 html-template-cards
├── docs/
│   └── ai-context.md         This file
└── releases/
    └── v1.0.0/
        └── card.yaml         Frozen snapshot of card.yaml at v1.0.0
```

Canonical editable file is the root `card.yaml`. `releases/vX.Y.Z/card.yaml` are immutable snapshots.

## Card architecture

- **Technology:** `custom:html-template-card` (PiotrMachowski's HTML Jinja2 Template card). Each card's `content` is a single long string of **inline CSS + HTML + Jinja2**.
- **Layout:** two `type: grid` sections, each with `background: {color: lightgray, opacity: 50}`, a `type: heading` card, then one `html-template-card` per reading.
- **One template per reading** — there is no shared template; the CSS `<style>` block is duplicated verbatim in every card. Editing the look means editing every card (7 places).
- **Gotchas:**
  - `ignore_line_breaks: true` is mandatory on every card — without it the multi-line string renders with stray `<br>`s.
  - The whole `content` is one quoted YAML string. Inner entity IDs use **single quotes** (`states('sensor.…')`); HTML attributes use **single quotes**; the inline `onclick` uses **escaped double quotes** (`\"`). Don't swap quote styles when editing.
  - Jinja runs **server-side** in the card; `{{ }}` / `{% %}` are evaluated before the HTML reaches the browser. Keep literal braces out of the CSS.
  - Tap handling is a raw DOM `onclick` dispatching a `hass-more-info` `CustomEvent` — not a card `tap_action`.

## Entity reference

| Entity | Reading | Unit (entity) | Tile |
|---|---|---|---|
| `sensor.pool_chem_temperature` | Temperature | °C | main |
| `sensor.pool_chem_free_chlorine` | Free chlorine | mg/L | main |
| `sensor.pool_chem_ph` | pH | pH | main |
| `sensor.pool_chem_orp` | ORP | mV | supporting |
| `sensor.pool_chem_salinity` | Salinity | ppm | supporting |
| `sensor.pool_chem_ec` | EC | µS/cm | supporting |
| `sensor.pool_chem_tds` | TDS | — | supporting |

Source device: Zigbee2MQTT, MQTT platform. Device id `e39c96e1b239cfb35bd1f3eab8b8a806`, config entry `0ead95fcc110f3671631d772b1a17775`, IEEE `0x781c9dfffe110488`. Hardware: Tuya BLE-YL01 (YK-S03 compatible — identical exposes).

## Card state logic

Scale, ideal setpoint and tolerance per reading (the `pct(val, min, max)` macro maps value → 0–100 % position; marker `left` is clamped to 5–95 %):

| Reading | Scale min–max | Ideal | Tolerance ± | Gradient |
|---|---|---|---|---|
| Temperature | 10 – 35 | 26 | 4 | `heat` (blue→red) |
| Free chlorine | 2.0 – 4.0 | 3.0 | 0.5 | `cent` (red→green→red) |
| pH | 6.6 – 7.8 | 7.2 | 0.3 | `cent` |
| ORP | 600 – 800 | 700 | 50 | `cent` |
| Salinity | 50 – 650 | 350 | 150 | `cent` |
| EC | 100 – 1100 | 600 | 250 | `cent` |
| TDS | 0 – 1100 | 500 | 300 | `cent` |

Label/verdict states per bar:

| State | Class | Condition | Label |
|---|---|---|---|
| Ideal | `ok` (cyan) | within ideal ± tolerance | `Ideal {{val}} <unit>` |
| Too High | `hi` (red) | above ideal + tolerance | `Too High …` |
| Too Low | `lo` (orange) | below ideal − tolerance | `Too Low …` |
| Error | `er` (grey) | `unavailable`/`unknown`, `0`, or `≥ 65535` (`≥ 6553` for FC) | `Error` |
| Recalc | `rc` (purple) | **free chlorine only:** pH invalid AND ORP valid | `Recalc {{rc_val}} ppm` |

**Button behaviour:** none — there are no buttons. The only interaction is tap-to-open-more-info on each bar.

## Key algorithms

- **Position mapping** — `pct(v, mn, mx) = clamp((v-mn)/(mx-mn)*100, 0, 100) | round(1)`; rendered marker uses `left: clamp(5%, {{p}}%, 95%)`.
- **Error sentinel** — Zigbee dropout / fault reports `0` or a `≥ 65535` (`0xFFFF`) sentinel; treated as Error. Free chlorine's broken-threshold is `≥ 6553`.
- **Free-chlorine recalc (when pH invalid, ORP valid):**
  ```
  rc_val = 10**((orp - 660) / 83)
           * (1 + 10**(7.2 - 7.53))
           / (1 + 10**(7.5 - 7.53))   |> round(2)
  ```
  Empirical Nernst-style fit; `7.2`/`7.5` bracket the assumed pH, `7.53`/`660`/`83` are fitted constants. Recalibrate against a test kit if needed.

## CSS class reference

| Class | Role |
|---|---|
| `.lz` | Outer wrapper; padding; carries the `onclick` more-info handler |
| `.lh` / `.ln` | Label header row / reading name text (with emoji) |
| `.bw` | Bar wrapper (relative, 12px tall, `margin-top:30px` to leave room for the floating label) |
| `.bar` | The gradient fill (absolute, inset) |
| `.cent` / `.heat` | Centred red→green→red gradient / blue→red heat gradient |
| `.mv` | Moving marker (white vertical line) |
| `.fb` | Floating label badge (Ideal/Too High/…) |
| `.fc` | Caret/triangle under the badge |
| `.ok` `.hi` `.lo` `.er` `.rc` | Badge colour variants (cyan/red/orange/grey/purple) |
| `.ok-c` `.hi-c` `.lo-c` `.er-c` `.rc-c` | Matching caret colours |
| `.sc` / `.sc .sp` | Scale-tick row / highlighted setpoint tick (green, bold) |

## Scripts & automations

None. This project is dashboard-only — no `automations.yaml`, no scripts, no helpers. (Hence no `automations.yaml` at root despite the general convention.)

## Build phases

| Phase | Work |
|---|---|
| 1 | Pair BLE-YL01 in Zigbee2MQTT; expose `sensor.pool_chem_*` |
| 2 | Main tile: temperature, free chlorine, pH gauge bars |
| 3 | Supporting tile: ORP, salinity, EC, TDS bars |
| 4 | Error-state handling + ORP→free-chlorine recalc fallback |
| 5 | Restructure into convention docs + GitHub repo (v1.0.0) |

## How to publish a new version

1. Edit root `card.yaml` (canonical).
2. Add a `## [X.Y.Z] — YYYY-MM-DD` section to `CHANGELOG.md` (Added/Changed/Removed/Fixed); move relevant items out of `[Unreleased]`.
3. Bump the README **Versions** table row.
4. Snapshot: `cp card.yaml releases/vX.Y.Z/card.yaml`.
5. Commit, tag `vX.Y.Z`, push branch + tags, and create the GitHub Release.
   Keep version numbers **only** in the README table, CHANGELOG, and `releases/` — never in install steps or feature prose.

## How to modify the card via Claude

- Changing look/behaviour means editing the duplicated `<style>`/logic in **all seven** cards in `card.yaml`. Preserve `ignore_line_breaks: true` and the quote conventions above.
- To retarget entities, replace the `pool_chem` prefix in every `states('sensor.pool_chem_…')`.
- After editing, validate by pasting into a Sections view and reading the rendered dashboard back.

## How to add the card via Claude (live HA)

1. Confirm the `html-template-card` resource is registered (list dashboard resources).
2. Resolve the user's actual pool-sensor entity IDs (don't assume `sensor.pool_chem_*`).
3. Fetch `card.yaml`, rewrite entity references, and insert both sections into the target view via the dashboard config API.
4. Read the dashboard back (or screenshot) to confirm render; expect **Error** bars until the sensor reports.
