# AGENTS.md — w4h-sample-icde-demo-csv

**Sample data package**: synthetic ICDE-demo CSVs + `manifest.yaml`, no code. Includes a raw geo signal (`gps_track`). Provenance and contents: [README.md](./README.md). The shared W4H architecture overview is at the end of this file.

## Where this repo fits in W4H

- **Layer:** contribution, `kind: dataset`, catalog slug `w4h-sample-icde-demo-csv`. The "sample" half of an ADR-016 pair; the importer is `w4h-sample-icde-demo-csv-importer`.
- **Why it exists alongside the Fitbit sample:** it covers what Fitbit cannot.
  - **Geo branch of GeoMTS** (ADR-008): `gps_track` maps `geo: { lat, lon }`; every Fitbit signal has `geo: null`.
  - **A different sampling interval**: every signal is `5min` versus Fitbit's `1min`. `w4h-sql-compute-mets` proves its METs template is interval-aware by running on both.
  - **More signal types**: distances, steps and sleep, in addition to heart rate, calories and weight.
- **Consumers:** the paired importer reads it from `../w4h-sample-icde-demo-csv` (or `W4H_SAMPLE_ICDE_DEMO_CSV_PATH`); `w4h-api`'s runner clones it next to the importer before an in-app run. It is **not** baked into `w4h-docker`.
- **Signal names here become table names:** `ds_<dataset_id>.icde_demo_<slug>`, with mappings in `w4h.tables`.
- **Provenance:** trimmed from `w4h-icde-demo` (ICDE 2024 demo paper) to 3 subjects × 3 days (2,592 rows per signal).

## Contents

| Path | What it is |
|---|---|
| `data/icde_demo_subjects.csv` | Subject id + observed window |
| `data/icde_demo_{heart_rate,calories,distances,steps,sleep,weight}.csv` | 5-minute series |
| `data/icde_demo_gps_track.csv` | Raw lat/lon (WGS84) trace |
| `manifest.yaml` | Columns, `geomts_mapping` per signal, `sampling_interval: 5min` |

**Invariant:** every signal file shares the same subjects and timestamps, so joins on `(subject, time)` work across signals. Keep it that way when editing.

## Change rules

| Change | Also update |
|---|---|
| CSV columns or new signal | `manifest.yaml`; the importer's dedupe keys and tests |
| Geo mapping of `gps_track` | Only with ADR-008 in mind; do not flatten geo into value columns |
| Interval or units of `calories` / `weight` | Re-check `w4h-sql-compute-mets` |

## Hard boundaries

- No code, API keys or credentials.
- Synthetic data only.

## Git expectations for agents

- Do not `git commit` or `git push` unless the user asks.
- Coordinate shape changes with `w4h-sample-icde-demo-csv-importer`.

<!-- BEGIN w4h-system-overview: identical in every W4H repo's AGENTS.md. Edit all copies together. -->

## W4H system overview (shared across all repos)

**W4H (Wearables for Health) Toolkit** loads wearable-device data into PostgreSQL, organizes it by team, project and dataset, controls access, and runs curated analyses on it. Each signal is a **GeoMTS** (Geospatial Multivariate Time Series) table whose roles (`subject`, `time`, optional `geo`, `values[]`) are stored in `w4h.tables.mapping_json`, not in fixed column names. Three repos form the platform, one packages it, and the rest are **contributions** listed in the in-app catalog (`w4h.contributions`).

| Repo | Role |
|---|---|
| `w4h-api` | Express API and Postgres DDL (`w4h` schema): auth, ACL, ingest, catalog, contribution runner, SQL engine. Port 2026 |
| `w4h-app` | Vue 3 + Vite SPA; talks only to `w4h-api`. Port 5173 |
| `w4h-docs` | Product rules, MVP, ADRs and guides; imported by the API as in-app Help. Owns product decisions |
| `w4h-docker` | Compose stack (`api`, `app`, `notebooks`) built from sibling repos; Postgres is external |
| `w4h-sample-fitbit-csv`, `w4h-sample-icde-demo-csv` | Synthetic sample CSVs plus `manifest.yaml` (ICDE includes a `gps_track` geo signal) |
| `w4h-sample-fitbit-csv-importer`, `w4h-sample-icde-demo-csv-importer` | Runnable importers (notebook, CLI, `run.sh`) for those samples |
| `w4h-fitbit-csv-file-importer` | Runnable importer for user-supplied Fitbit CSVs: profile headers, review a drafted mapping, ingest |
| `w4h-sql-compute-mets` | SQL contribution (YAML + SQL template) that `w4h-api` executes in-process |
| `fft_approximate` | Postgres C extension, catalog slug `fft-approximate`; no API involvement |
| `w4h-demo-approximate-queries` | Notebook comparing `fft_approximate` with exact averages over direct Postgres |

**Main flows:**

1. **Ingest:** importers post 500-row chunks to `POST /datasets/:id/import/csv-batch`. `lib/ingest-csv.js` creates `ds_<dataset_id>.<table_prefix>_<signal>` and upserts its `w4h.tables` mapping.
2. **In-app Run (ADR-022):** `POST /contributions/:slug/run` makes `lib/contribution-runner.js` spawn `../<slug>/run.sh` with the user's `W4H_API_KEY`, `W4H_API_BASE` and `W4H_DATASET_ID`. Stdout lines `@@PROGRESS@@ {json}` drive the progress bar.
3. **SQL analytics (ADR-023):** `lib/sql-contribution-engine.js` reads `../<slug>/manifest.yaml` and `sql/`, binds identifiers from `w4h.tables`, and runs in-process. There is no free-SQL endpoint.
4. **Extensions:** the operator installs `fft_approximate` on their Postgres; notebooks call it directly.

**Layout:** clone every repo as a sibling in one folder. The API finds contributions at `../<slug>` (override `CONTRIBUTION_RUNNER_REPO_ROOT`) and docs at `DOCS_REPO_PATH=../w4h-docs`.

**Cross-repo changes:** `w4h-api` first (regenerate `API_INDEX.md`), then `w4h-app`, then `w4h-docs`. Catalog rows are seeded in `w4h-api/data/schema.sql`. Start at `w4h-docs/dev/mvp.md` and `w4h-docs/dev/ADR-README.md` for decisions, and `w4h-docs/dev/developer-setup.md` for local setup, getting an API key, test personas, troubleshooting, per-repo test commands, default branches and a glossary. Do not commit or push unless asked; never commit `.env` files, API keys or participant data.

<!-- END w4h-system-overview -->
