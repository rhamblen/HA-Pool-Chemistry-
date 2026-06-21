# Installation

**Repository:** `https://github.com/rhamblen/HA-Pool-Chemistry-`

The Pool Chemistry tiles are pure dashboard cards — there are **no helpers, scripts or automations to create**. The only moving parts are one HACS custom card and the pool sensor's entities.

---

## Option A — Claude-assisted (recommended)

If you use Claude Code / the Claude Home Assistant integration, just say:

> "Add the Pool Chemistry tiles from `https://github.com/rhamblen/HA-Pool-Chemistry-` to my Home Assistant. Confirm the HTML Jinja2 Template card is installed, find my pool sensor's entities, add both the *main* and *supporting* sections to my pool dashboard view, and verify they render."

Claude will:

1. Check that the **HTML Jinja2 Template card** resource is registered (and tell you how to install it via HACS if not).
2. Locate your pool tester's entities and confirm their entity IDs (these may not be `sensor.pool_chem_*` on your system).
3. Fetch [`card.yaml`](card.yaml) and rewrite the `states('sensor.pool_chem_…')` references to match your entities.
4. Insert both grid sections into the dashboard view you choose.
5. Read the rendered dashboard back and confirm the bars appear (showing **Error** until the sensor reports).

---

## Option B — Manual

### Prerequisites

| Requirement | Detail |
|---|---|
| Home Assistant | Recent version with **Sections** (grid) dashboard views |
| HACS | Installed and working |
| HTML Jinja2 Template card | [`PiotrMachowski/Home-Assistant-Lovelace-HTML-Jinja2-Template-card`](https://github.com/PiotrMachowski/Home-Assistant-Lovelace-HTML-Jinja2-Template-card) |
| Pool sensor | Tuya **BLE-YL01** or **YK-S03** via Zigbee2MQTT |

### Step 1 — Install the custom card

1. HACS → **Frontend** → search **"HTML Jinja2 Template card"** → install.
2. Confirm the resource is registered: **Settings → Dashboards → ⋮ → Resources** should list
   `/hacsfiles/Home-Assistant-Lovelace-HTML-Jinja2-Template-card/html-template-card.js` (type *module*).
3. Hard-refresh your browser.

> If the card is missing you'll see *"Custom element doesn't exist: html-template-card"*.

### Step 2 — Provide the sensor entities

This card consumes seven sensor entities — it does **not** create any helpers.

1. Pair your Tuya **BLE-YL01** / **YK-S03** in **Zigbee2MQTT**.
2. Name the device **Pool Chem** so Zigbee2MQTT generates these entity IDs:

   | Property (Z2M) | Entity ID | Unit |
   |---|---|---|
   | `temperature` | `sensor.pool_chem_temperature` | °C |
   | `free_chlorine` | `sensor.pool_chem_free_chlorine` | mg/L |
   | `ph` | `sensor.pool_chem_ph` | pH |
   | `orp` | `sensor.pool_chem_orp` | mV |
   | `salinity` | `sensor.pool_chem_salinity` | ppm |
   | `ec` | `sensor.pool_chem_ec` | µS/cm |
   | `tds` | `sensor.pool_chem_tds` | ppm |

   If you name the device anything else, note the resulting prefix and adjust the entity references in Step 3.

### Step 3 — Add the card

1. Download the current card config from the stable root file:
   [`card.yaml`](https://raw.githubusercontent.com/rhamblen/HA-Pool-Chemistry-/main/card.yaml)
2. Open your pool dashboard → **Edit dashboard** → the **Pool** view → **Edit in YAML**.
3. Paste the two entries from `card.yaml` into the view's `sections:` list (or add each `custom:html-template-card` individually).
4. If your entities are **not** `sensor.pool_chem_*`, find-and-replace `pool_chem` with your prefix throughout the pasted YAML.
5. Save.

### Step 4 — Verify

You should see two grid tiles:

- **Pool Chemistry (main)** — Temperature, Free Chlorine, pH.
- **Pool Chemistry (supporting)** — ORP, Salinity, EC, TDS.

Each bar shows a gradient scale with a tick at the ideal value. Until the sensor reports, every bar correctly shows a grey **Error** label — that is expected, not a fault. Tapping a bar opens the entity's history dialog.

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| *"Custom element doesn't exist: html-template-card"* | Card not installed or resource not registered | Install via HACS (Step 1), confirm the resource, hard-refresh |
| Every bar shows **Error** (grey) | Sensor `unavailable`/`unknown`, or device offline/broken | Check the Zigbee2MQTT device is paired and reporting; the bars recover automatically |
| Bars render but all read the wrong entity | Entity prefix doesn't match | Replace `pool_chem` with your device's prefix in the card YAML (Step 3.4) |
| Free chlorine shows **Recalc** (purple) | pH is invalid but ORP is valid — value is estimated from ORP | Fix/clean the pH probe; the measured value returns once pH is valid |
| Labels run off the end of a bar | — | Not expected: marker position is clamped to 5–95 %; re-check you pasted the full card content |
| Tiles appear as a single line of broken HTML | `ignore_line_breaks: true` was dropped when editing | Restore `ignore_line_breaks: true` on each `html-template-card` |
