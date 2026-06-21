# Pool Chemistry Tiles

Documentation for the two **Pool Chemistry** cards on the *Pool* tab of the **My Home** dashboard in Home Assistant.

These tiles render each chemistry reading as a horizontal "gauge" bar with a coloured scale, a moving marker, a floating label (Ideal / Too High / Too Low / Error / Recalc), and an ideal-value tick highlighted in green. Tapping any bar opens the entity's more-info dialog.

---

## 1. Where they live

| | |
|---|---|
| **Dashboard** | My Home (`url_path: my-home`) |
| **View / tab** | *Pool* (view index 6) |
| **Tile 1** | Section 0 — heading **"Pool Chemistry (main)"** |
| **Tile 2** | Section 1 — heading **"Pool Chemistry (supporting)"** |

Both sections are `grid` cards with a `lightgray` / 50% background, a `heading` card, then one `custom:html-template-card` per reading.

---

## 2. Required dependencies

### 2.1 Custom card (HACS)

The tiles are built entirely from the **HTML Jinja2 Template card**:

- **Card type:** `custom:html-template-card`
- **HACS repo:** `PiotrMachowski/Home-Assistant-Lovelace-HTML-Jinja2-Template-card`
- **Resource URL (already registered):** `/hacsfiles/Home-Assistant-Lovelace-HTML-Jinja2-Template-card/html-template-card.js`

> If this card is missing the tiles show "Custom element doesn't exist: html-template-card". Install it via HACS → Frontend.

### 2.2 No helpers required

There are **no `input_*` helpers, template sensors, or utility meters** behind these tiles. All thresholds, scales, ideal values, and error logic are **hard-coded inside the card templates**. The only inputs are the raw sensor entities below.

### 2.3 Supported hardware

Designed for the **Tuya BLE-YL01** 7-in-1 pool tester via Zigbee2MQTT
([z2m device page](https://www.zigbee2mqtt.io/devices/BLE-YL01.html)).

It is **also compatible with the Tuya YK-S03**
([z2m device page](https://www.zigbee2mqtt.io/devices/YK-S03.html)) — the two
are effectively the same hardware under different model IDs and expose an
**identical** set of properties/units. No template changes are needed:

| Property used by tiles | BLE-YL01 | YK-S03 |
|---|---|---|
| `temperature` (°C) | ✅ | ✅ |
| `ph` (pH) | ✅ | ✅ |
| `free_chlorine` (mg/L) | ✅ | ✅ |
| `orp` (mV) | ✅ | ✅ |
| `salinity` (ppm) | ✅ | ✅ |
| `ec` (µS/cm) | ✅ | ✅ |
| `tds` (ppm) | ✅ | ✅ |

Both also expose `battery` and writable max/min threshold properties, which the
tiles don't use.

> **The only thing that varies is the entity prefix.** Zigbee2MQTT derives the
> entity IDs from the device's friendly name. These tiles expect
> `sensor.pool_chem_*`, so name the device **"Pool Chem"** in Zigbee2MQTT — or
> update the seven `states('sensor.pool_chem_…')` references in the YAML to
> match your device's entity IDs.

### 2.4 Source sensors

All seven entities come from a **Zigbee2MQTT** pool sensor device (MQTT platform), not from HA helpers:

| Card label | Entity ID | Unit (entity) | Source |
|---|---|---|---|
| 🌡️ Temperature | `sensor.pool_chem_temperature` | °C | Zigbee2MQTT |
| 🧪 Free Chlorine | `sensor.pool_chem_free_chlorine` | mg/L | Zigbee2MQTT |
| ⚗️ pH | `sensor.pool_chem_ph` | pH | Zigbee2MQTT |
| ⚡ ORP | `sensor.pool_chem_orp` | mV | Zigbee2MQTT |
| 🧂 Salinity | `sensor.pool_chem_salinity` | ppm | Zigbee2MQTT |
| 〰️ EC | `sensor.pool_chem_ec` | µS/cm | Zigbee2MQTT |
| 💧 TDS | `sensor.pool_chem_tds` | ppm | Zigbee2MQTT |

- **Device id:** `e39c96e1b239cfb35bd1f3eab8b8a806`
- **MQTT config entry:** `0ead95fcc110f3671631d772b1a17775`
- **Zigbee IEEE:** `0x781c9dfffe110488` (unique_id suffix `_zigbee2mqtts`)

> ⚠️ At time of documenting, all seven sensors read `unavailable` (`restored: true`) — i.e. the Zigbee device was not reporting. The tiles handle this with the **Error** state (see below). Bring the sensor online to see live values.

---

## 3. Thresholds & scales (the "config")

Each bar maps its sensor onto a fixed min–max scale. The **ideal** value (green tick) has a ± tolerance window that shows "Ideal"; outside it shows "Too High" / "Too Low".

### Tile 1 — Pool Chemistry (main)

| Reading | Scale (min–max) | Ideal | Tolerance (green band) | Scale ticks |
|---|---|---|---|---|
| Temperature | 10 – 35 °C | 26 °C | ±4 (22–30) | 10 · 18 · 22 · **26** · 30 · 35 |
| Free Chlorine | 2.0 – 4.0 ppm | 3.0 ppm | ±0.5 (2.5–3.5) | 2.0 · 2.5 · **3.0** · 3.5 · 4.0 |
| pH | 6.6 – 7.8 | 7.2 | ±0.3 (6.9–7.5) | 6.8 · 7.0 · **7.2** · 7.4 · 7.6 |

Temperature uses a blue→red **heat** gradient; Free Chlorine and pH use a red→green→red **centred** gradient.

### Tile 2 — Pool Chemistry (supporting)

| Reading | Scale (min–max) | Ideal | Tolerance (green band) | Scale ticks |
|---|---|---|---|---|
| ORP | 600 – 800 mV | 700 mV | ±50 (650–750) | 600 · 650 · **700** · 750 · 800 |
| Salinity | 50 – 650 ppm | 350 ppm | ±150 (200–500) | 50 · 200 · **350** · 500 · 650 |
| EC | 100 – 1100 µS/cm | 600 µS/cm | ±250 (350–850) | 100 · 350 · **600** · 850 · 1100 |
| TDS | 0 – 1100 | 500 | ±300 (200–800) | 0 · 200 · **500** · 800 · 1100 |

All four use the **centred** gradient.

---

## 4. Behaviour / logic notes

- **Error detection** — a reading is shown as **"Error"** (grey) when the raw state is `unavailable` / `unknown`, equals `0`, or is `≥ 65535` (Free Chlorine uses `≥ 6553`). These catch a disconnected or faulted Zigbee sensor reporting a sentinel value.
- **Tap to open** — each bar fires a `hass-more-info` event for its entity, so tapping opens the standard HA detail/history dialog.
- **Free-chlorine ORP fallback (smart recalc)** — if **pH is unavailable** but **ORP is valid**, the Free Chlorine bar *estimates* FC from ORP using a Nernst-style relation and shows a purple **"Recalc"** label instead of the measured value:

  ```
  rc_val = 10^((orp - 660) / 83) × (1 + 10^(7.2 - 7.53)) / (1 + 10^(7.5 - 7.53))
  ```

  When pH is valid, the measured `sensor.pool_chem_free_chlorine` value is used directly.
- **Marker position** is clamped to 5–95 % so labels never run off the ends of the bar.

> **Unit-label note:** the card labels Free Chlorine as `ppm` (entity unit is `mg/L` — numerically equivalent for pool water) and TDS as `g/L` in the "Ideal" label (entity unit is `ppm`). These are cosmetic label strings in the templates, not conversions. Flagged here in case you want to align them.

---

## 5. Recreating / restoring the tiles

The full Lovelace YAML for both sections is in **[`pool-chemistry-tiles.yaml`](pool-chemistry-tiles.yaml)**. To restore:

1. Ensure the HTML Jinja2 Template card (§2.1) is installed.
2. Edit the *Pool* view of the **My Home** dashboard → *Edit in YAML*.
3. Paste the two `sections` entries (or the individual cards) into the view.

The sensor entity IDs must match (`sensor.pool_chem_*`). If your Zigbee2MQTT device exposes different entity IDs, update the `states('sensor.pool_chem_…')` references in each card.
