# Changelog

All notable changes to this project will be documented in this file.

The format is based on Keep a Changelog and this project currently follows a simple manual versioning approach.

## [Unreleased]

No unreleased changes.


## [0.1.1] - 2026-04-08

### Added

- Dashboard variable `top_n` for configurable Top-N identities in `Top Senders` and `Top Recipients`.
- Dashboard variable `top_n_results` for configurable Top-N buckets in `Results`.
- Optional Grafana feature toggles in Docker Compose for scene-based dashboard capabilities:
  - `dashboardScene`
  - `kubernetesDashboards`

### Changed

- `README.ru.md` wording was refined for clearer project audience description.
- Dashboard lower and middle rows were compacted to a uniform panel height for better viewport fit.
- `Top Senders` and `Top Recipients` terms aggregations now use `${top_n:raw}`.
- `Results` terms aggregation now uses `${top_n_results:raw}`.

### Validated

- Dashboard JSON remains syntactically valid after layout and variable updates.
- Grafana container restart completed successfully and `/api/health` returned `database: ok`.
- Quick lab run with 100 messages completed successfully with `send_status=sent` for all messages.


## [0.1.0] - 2026-04-08

### Added

- Docker Compose stack for Grafana OSS.
- Grafana provisioning for Elasticsearch data sources:
  - `Kerio Mail Flows` backed by `kerio-flow-*`.
  - `Kerio Raw Events` backed by `kerio-connect-*`.
- `Kerio Mail Traffic` dashboard for Kerio Connect mail load-test visualization.
- Standard project documentation based on the latest `/root/README_TEMPLATE.md` and `/root/README_TEMPLATE.ru.md`.
- Russian README aligned with the English README for lab onboarding.
- `.env.example` with non-secret placeholder values for Grafana and Elasticsearch configuration.
- `LICENSE` with the Apache 2.0 license text.

### Changed

- `.gitignore` now excludes `.env` and local environment variants so runtime secrets stay out of git.
- Dashboard layout was compacted and then resized for better visibility of top-line stat panels, time series, and lower-row breakdowns.
- Dashboard queries use `${run_id:raw}` so `run_id=*` works as a wildcard for all `KT-*` test traffic.

### Validated

- Grafana container health endpoint returned `database: ok`.
- Both Elasticsearch data sources returned healthy status through the Grafana API.
- Live Kerio mail test traffic populated both raw and aggregated Elasticsearch indexes.
