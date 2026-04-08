# Handoff

## Purpose

This file captures the current working state of `kerio-grafana-dashboard` so work can continue quickly in another chat or shell session.

## Current Snapshot

- Updated: 2026-04-08
- Repository path: `/root/kerio-grafana-dashboard`
- GitHub repository: `https://github.com/foksk76/kerio-grafana-dashboard` after initial publication
- Runtime Grafana host: `10.4.29.72`
- Grafana URL: `http://10.4.29.72:3000/`
- Dashboard URL: `http://10.4.29.72:3000/d/kerio-mail-traffic/kerio-mail-traffic`
- Elasticsearch endpoint used by the current lab `.env`: `http://10.4.29.70:9200`
- Dashboard UID: `kerio-mail-traffic`
- Primary data source UID: `kerio-flow-es`
- Raw event data source UID: `kerio-connect-es`

## Stack State

Grafana stack:

- `kerio-grafana`: running and healthy during the last validation.
- Docker image: `grafana/grafana-oss:12.4.2`
- Published port: `3000`

Expected upstream ELK stack:

- Elasticsearch reachable from Grafana at `ELASTICSEARCH_URL`.
- `kerio-flow-*` contains aggregated mail-flow documents.
- `kerio-connect-*` contains raw Kerio Connect parsed events.

## Latest Validation

Validated on 2026-04-08:

- Grafana `/api/health` returned `database: ok`.
- Grafana data source health:
  - `kerio-flow-es`: `OK`
  - `kerio-connect-es`: `OK`
- Dashboard model checks:
  - `Run Filter` panel removed.
  - Top-line panels are `stat` panels with `graphMode=area`.
  - Dashboard total grid height is `24`.
  - Dashboard queries use `${run_id:raw}`.

Recent live run IDs:

- `GRAFANA-DASHBOARD-20260408-120340` - completed verification run, `planned_messages=40`, `passed=40`, `failed=0`, `unparsed_hits=0`.
- `GRAFANA-DYNAMICS-20260408-122042` - interrupted live dynamics run; Elasticsearch still contains partial data from the run.

## Suggested Resume Commands

```bash
cd /root/kerio-grafana-dashboard
docker compose ps
curl -sS http://127.0.0.1:3000/api/health
```

Check data source health:

```bash
cd /root/kerio-grafana-dashboard
set -a
. ./.env
set +a

curl -sS -u "$GRAFANA_ADMIN_USER:$GRAFANA_ADMIN_PASSWORD" \
  http://127.0.0.1:${GRAFANA_HTTP_PORT:-3000}/api/datasources/uid/kerio-flow-es/health

curl -sS -u "$GRAFANA_ADMIN_USER:$GRAFANA_ADMIN_PASSWORD" \
  http://127.0.0.1:${GRAFANA_HTTP_PORT:-3000}/api/datasources/uid/kerio-connect-es/health
```

Reload Grafana provisioning after dashboard edits:

```bash
cd /root/kerio-grafana-dashboard
python3 -m json.tool dashboards/kerio-mail-traffic.json >/tmp/kerio-mail-traffic.json.check
docker compose restart grafana
```

## Notes

- `.env` contains runtime secrets and should not be committed.
- Use `.env.example` for the public repository.
- The repository uses Apache License 2.0 to match the adjacent `kerio-logstash-project` licensing style.
