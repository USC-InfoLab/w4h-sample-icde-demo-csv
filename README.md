# w4h-sample-icde-demo-csv

Synthetic **ICDE demo** CSV sample package for the W4H Toolkit (data + `manifest.yaml` only). Import tooling lives in the paired repo **[w4h-sample-icde-demo-csv-importer](https://github.com/USC-InfoLab/w4h-sample-icde-demo-csv-importer)**.

**Publisher:** usc-infolab · Naming: `w4h-sample-<vendor>-<format>` ([ADR-016](https://github.com/USC-InfoLab/w4h-docs/blob/main/dev/ADR-016-paired-sample-importer-repos.md)).

Unlike **[w4h-sample-fitbit-csv](https://github.com/USC-InfoLab/w4h-sample-fitbit-csv)** (no GPS — real Fitbit CSV exports don't carry location), this package includes a raw geo signal (`gps_track`) so importer and GeoMTS tooling can be exercised against the geo branch of the mapping, not just the non-geo one. See [ADR-008](https://github.com/USC-InfoLab/w4h-docs/blob/main/dev/ADR-008-geomts-data-model.md).

## Provenance

Trimmed from **[w4h-icde-demo](https://github.com/USC-InfoLab/w4h-icde-demo)**'s `data/synthetic_timeseries_data.csv` — the synthetic dataset behind the ICDE 2024 demo paper (17 subjects x ~2 months at 5-minute resolution) — down to **3 subjects x 3 days** (2,592 rows/signal) so a full import runs quickly against a dev API. Every signal file, including `gps_track`, shares the same subjects and timestamps, so cross-signal joins on `(subject, time)` work.

That prototype stored location as a separate PostGIS `locations` table joined by `(user_id, timestamp)`, not as columns on every row — this sample mirrors that split with a dedicated `gps_track` signal. See ADR-008's "Precedent" section for detail.

## How to use

1. Clone this repo next to `w4h-sample-icde-demo-csv-importer`.
2. Open **Contributions → ICDE demo CSV importer** in the W4H app (or this repo's sibling README) for notebook and CLI steps.
3. **Admins** ingest into their team catalog with a personal API key. **Users** use the importer notebook against data they already have access to; do not re-ingest **W4H Samples**.

## Contents

| File | Description |
|------|-------------|
| `data/icde_demo_subjects.csv` | Subject id and observed window (`start_date`, `end_date`) within this trimmed sample |
| `data/icde_demo_heart_rate.csv` | Simulated heart rate (bpm), 5-minute resolution |
| `data/icde_demo_calories.csv` | Simulated calories burned, 5-minute resolution |
| `data/icde_demo_distances.csv` | Simulated distance traveled, 5-minute resolution |
| `data/icde_demo_steps.csv` | Simulated step count, 5-minute resolution |
| `data/icde_demo_sleep.csv` | Simulated sleep signal value, 5-minute resolution |
| `data/icde_demo_weight.csv` | Simulated weight — continuous synthetic stream, not a realistic weigh-in cadence |
| `data/icde_demo_gps_track.csv` | Raw GPS trace (lat/lon, WGS84) — the geo signal this package exists to exercise |

Column definitions, units, and GeoMTS mapping (including `gps_track`'s `geo: {lat, lon}` role mapping) are in [`manifest.yaml`](manifest.yaml).

## Known data-quality fixtures

Carried over from the source dataset on purpose, not manufactured for this sample (parallel to `w4h-sample-fitbit-csv`'s deliberate fixtures — not flagged in `manifest.yaml`, which is the vendor mapping, not an answer key):

- **Malformed geo encoding** — `data/icde_demo_gps_track.csv`, subjects `4c9fe48e` and `84d5380c` (all rows, all 3 days): `latitude` holds a Python tuple repr, e.g. `"(34.0224, -118.2851)"`, with `longitude` empty — a real bug in the `w4h-icde-demo` synthetic data generator, reproduced as-is. Subject `c12574be` is unaffected (clean `latitude`/`longitude` floats). This is a structural mismatch the importer's vendor-spec check catches at import time (`type_mismatch` on `latitude`) — see the paired [importer repo](https://github.com/USC-InfoLab/w4h-sample-icde-demo-csv-importer)'s test suite.

Do not "fix" these rows — if you add a new deliberate fixture, document it here, not in `manifest.yaml`.

## License

MIT — see [LICENSE](LICENSE). Synthetic sample data only; no real PHI. USC coordinates in `gps_track` are the same illustrative center point used by the `w4h-icde-demo` prototype, not real participant locations.
