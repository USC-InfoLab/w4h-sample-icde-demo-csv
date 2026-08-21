# AGENTS.md — w4h-sample-icde-demo-csv

Sample **ICDE demo CSV** data package (no import code). Includes a raw geo signal (`gps_track`) — the counterpart to `w4h-sample-fitbit-csv`, which has none.

| Repo | Role |
|------|------|
| **w4h-sample-icde-demo-csv** (this repo) | CSV files + `manifest.yaml` |
| **w4h-sample-icde-demo-csv-importer** | Notebook + CLI + tests |

Do not add API keys or database credentials here. Import uses the paired importer with `W4H_API_KEY` in local `.env`.
