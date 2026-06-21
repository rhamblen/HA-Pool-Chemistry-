# Pool Chemistry Tiles

Two Home Assistant dashboard tiles that turn a Tuya 7-in-1 pool tester into an at-a-glance water-chemistry panel. Each reading — temperature, free chlorine, pH, ORP, salinity, EC and TDS — is drawn as a coloured gauge bar with a moving marker and a plain-language verdict (**Ideal / Too High / Too Low**), so you can see what your pool needs without reading numbers.

> Built around the [Tuya BLE-YL01](https://www.zigbee2mqtt.io/devices/BLE-YL01.html) pool sensor (also works with the [Tuya YK-S03](https://www.zigbee2mqtt.io/devices/YK-S03.html)) exposed through Zigbee2MQTT.

---

## Features

- **Seven-in-one panel** — temperature, free chlorine, pH, ORP, salinity, EC and TDS, split across a *main* tile and a *supporting* tile.
- **Gauge bars, not numbers** — every reading maps onto a fixed colour scale with a moving marker and a green ideal-tick, readable in a glance.
- **Plain-language verdicts** — each bar labels itself **Ideal**, **Too High** or **Too Low** against a target ± tolerance band.
- **Robust error handling** — a disconnected or faulted sensor shows a grey **Error** bar instead of a misleading `0`.
- **pH-aware chlorine** — if pH is missing or out of range, free chlorine is re-estimated from ORP and flagged **Recalc**, so the reading is never silently wrong.
- **No helpers required** — all thresholds and logic live inside the card; there are no `input_*` helpers, template sensors or YAML packages to maintain.
- **Tap for history** — tapping any bar opens the entity's more-info dialog.

---

## Versions

| Version | Status | Features |
|---|---|---|
| 0.1.0 | ✅ Released | Main + supporting tiles, 7 gauge bars, error states, ORP→chlorine recalc fallback |
| — | 🔜 Planned | Optional `input_number` helpers for editable targets; battery & signal badges |

See the [CHANGELOG](CHANGELOG.md) for details.

---

## Prerequisites

| Requirement | Detail |
|---|---|
| Home Assistant | Recent version with **Sections** (grid) dashboard views |
| HACS | Used to install the custom card below |
| HTML Jinja2 Template card | [`PiotrMachowski/Home-Assistant-Lovelace-HTML-Jinja2-Template-card`](https://github.com/PiotrMachowski/Home-Assistant-Lovelace-HTML-Jinja2-Template-card) — provides `custom:html-template-card` |
| Pool sensor | Tuya **BLE-YL01** or **YK-S03**, paired through Zigbee2MQTT |

---

## Installation

Two paths — full detail in **[INSTALLATION.md](INSTALLATION.md)**.

### Option A — Claude-assisted (recommended)

Tell Claude (with this repo open or its URL to hand):

> "Add the Pool Chemistry tiles from `https://github.com/rhamblen/HA-Pool-Chemistry-` to my Home Assistant. Check the HTML Jinja2 Template card is installed, then add both sections to my pool dashboard and point them at my pool sensor's entities."

Claude will check the custom card, confirm your `sensor.*` entity names, insert both tiles into your chosen view, and verify they render.

### Option B — Manual

1. Install the **HTML Jinja2 Template card** via HACS.
2. Pair your pool tester in Zigbee2MQTT and name it **Pool Chem** (this yields the `sensor.pool_chem_*` entities).
3. Copy [`card.yaml`](card.yaml) into a Sections view and adjust the entity names if needed.

Step-by-step instructions, helper-free setup and troubleshooting are in **[INSTALLATION.md](INSTALLATION.md)**.

---

## Entities used

| Entity | Role |
|---|---|
| `sensor.pool_chem_temperature` | Water temperature gauge (main tile) |
| `sensor.pool_chem_free_chlorine` | Free chlorine gauge (main); falls back to an ORP estimate when pH is invalid |
| `sensor.pool_chem_ph` | pH gauge (main); also gates the chlorine recalc |
| `sensor.pool_chem_orp` | ORP gauge (supporting); source value for the chlorine recalc |
| `sensor.pool_chem_salinity` | Salinity gauge (supporting) |
| `sensor.pool_chem_ec` | Electrical conductivity gauge (supporting) |
| `sensor.pool_chem_tds` | Total dissolved solids gauge (supporting) |

The entity prefix (`pool_chem`) comes from the device's friendly name in Zigbee2MQTT — rename the device or edit the `states('sensor.pool_chem_…')` references to match your setup.

---

## How it works

```
 Tuya BLE-YL01 / YK-S03  (pool tester)
            │  Zigbee
            ▼
        Zigbee2MQTT  ──MQTT──►  sensor.pool_chem_*
            │                   (temperature, ph, free_chlorine,
            │                    orp, salinity, ec, tds)
            ▼
   custom:html-template-card   (one per reading)
        │
        ├─ read raw state ─────► invalid? ──► grey  "Error"
        ├─ map value onto a fixed min–max colour scale
        ├─ classify vs target ± tolerance ──► Ideal / Too High / Too Low
        └─ free chlorine only:
              pH invalid + ORP valid ──► estimate from ORP ──► "Recalc"
        │
        ▼
   gauge bar:  gradient  +  moving marker  +  floating label  +  ideal tick
```

Each bar is a self-contained Jinja2 template inside an `html-template-card`. It reads one sensor, decides whether the value is valid, positions a marker along a colour gradient, and prints a verdict. The free-chlorine bar additionally watches pH and ORP so it can stay useful when the pH probe drops out. No automations or helpers are involved — the card is the entire logic.

---

## Related projects

None yet. (Companion automations/notifications could live in a sibling repo later.)

## Changelog

See [CHANGELOG.md](CHANGELOG.md).

## Licence

[MIT](LICENSE) © 2026 Richard Hamblen.
