# v0.1.0 — Pool Chemistry Tiles

First documented release. Two Home Assistant dashboard tiles that visualise a Tuya 7-in-1 pool tester (BLE-YL01 / YK-S03) as colour-coded gauge bars.

## Added
- **Pool Chemistry (main)** tile — temperature, free chlorine, pH.
- **Pool Chemistry (supporting)** tile — ORP, salinity, EC, TDS.
- Per-reading colour scales, ideal setpoints and ± tolerance bands.
- **Error** state for `unavailable`/`unknown`/`0`/sentinel readings.
- **ORP→free-chlorine recalc** fallback when pH is invalid but ORP is valid.
- Tap-to-open more-info on each bar.
- `card.yaml` (canonical) plus full docs: `README.md`, `INSTALLATION.md`, `docs/ai-context.md`.

## Notes
- Built for the Tuya BLE-YL01; verified compatible with the Tuya YK-S03 (identical exposes).
- No `input_*` helpers, scripts or automations — all logic lives in the card templates.
- At release the physical sensor was offline, so all bars render the **Error** state by design.

See [CHANGELOG.md](CHANGELOG.md) for the full entry.
