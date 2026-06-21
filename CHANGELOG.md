# Changelog

All notable changes to this project are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Planned
- Optional `input_number` helpers for editable target/tolerance values per reading.
- Battery and Zigbee signal badges sourced from the device's `battery`/`linkquality`.
- Align cosmetic unit labels with entity units (free chlorine `ppm` vs `mg/L`; TDS `g/L` vs `ppm`).

---

## [1.0.0] — 2026-06-21

First documented release. Two dashboard tiles on the **My Home → Pool** view.

### Added
- **Pool Chemistry (main)** tile — gauge bars for `sensor.pool_chem_temperature`, `sensor.pool_chem_free_chlorine`, `sensor.pool_chem_ph`.
- **Pool Chemistry (supporting)** tile — gauge bars for `sensor.pool_chem_orp`, `sensor.pool_chem_salinity`, `sensor.pool_chem_ec`, `sensor.pool_chem_tds`.
- Per-reading colour scales, ideal setpoints and ± tolerance bands (see `docs/ai-context.md` for the full table).
- **Error** state for any reading that is `unavailable`/`unknown`, `0`, or `≥ 65535` (`≥ 6553` for free chlorine).
- **ORP→free-chlorine recalc fallback** — when `sensor.pool_chem_ph` is invalid but `sensor.pool_chem_orp` is valid, free chlorine is estimated from ORP and shown as **Recalc**.
- Tap-to-open: each bar dispatches a `hass-more-info` event for its entity.
- `card.yaml` — full Lovelace YAML for both sections.
- Documentation set: `README.md`, `INSTALLATION.md`, `docs/ai-context.md`.

### Notes
- Hardware: built for the Tuya **BLE-YL01**; verified entity/property compatibility with the Tuya **YK-S03** (identical exposes).
- No `input_*` helpers, template sensors, scripts or automations are used — all logic is inside the card templates.
- At time of release the physical sensor was offline, so all bars render the **Error** state; this is expected behaviour.

### Design decisions
- Thresholds are hard-coded in each template (no helpers) to keep the card self-contained and copy-paste portable.
- The free-chlorine recalc uses an empirical Nernst-style fit (`10**((orp-660)/83) * (1+10**(7.2-7.53)) / (1+10**(7.5-7.53))`) so the panel stays useful when the pH probe drops out.
