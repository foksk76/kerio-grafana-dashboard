Language: [English](README.md) | [Русский](README.ru.md)

# kerio-grafana-dashboard

Grafana provisioning for Kerio Connect mail traffic stored in Elasticsearch.

> **Project status:** Lab-friendly dashboard project for safely checking Kerio Connect mail-flow visibility before production use.

> **Language policy:** `README.md` is the main English README. `README.ru.md` is the main Russian translation for lab work and quick onboarding. Keep the language switcher as the first line in both files.

## Why this repository exists

Kerio Connect mail logs become much easier to operate when parsed mail events and aggregated Queue-ID flows can be viewed as live traffic charts.

The lab goal is to measure and display production-style mail-flow indicators for monitoring, anomaly detection, and later transfer of the approach to a production environment.

This repository provides a small, reproducible Grafana layer for the Kerio Connect logging lab. It configures Elasticsearch data sources and a mail-traffic dashboard in Grafana. The dashboard helps engineers inspect mail-run dynamics and compare them with parameters set by the load-generation script.

## Project family

This repository is part of the **Kerio Connect Monitoring & Logging** project family:

1. [kerio-connect](https://github.com/foksk76/kerio-connect) - reproducible Kerio Connect lab environment
2. [kerio-logstash-project](https://github.com/foksk76/kerio-logstash-project) - parsing, normalization, and validation pipeline for Kerio syslog in ELK
3. [kerio-syslog-anonymizer](https://github.com/foksk76/kerio-syslog-anonymizer) - deterministic anonymization of real log data for safe public use and repeatable correlation
4. `kerio-grafana-dashboard` - Grafana provisioning and dashboard layer for Kerio mail traffic in Elasticsearch

## Where this repository fits

```text
Kerio Connect -> Syslog (RFC5424) -> Logstash -> Elasticsearch -> Grafana
```

- `kerio-connect` provides the Kerio Connect lab host.
- `kerio-logstash-project` parses Kerio syslog and writes `kerio-connect-*` and `kerio-flow-*` indexes.
- `kerio-syslog-anonymizer` prepares real logs for safe sharing when needed.
- This repository starts Grafana and provisions the Kerio mail traffic dashboard.

## Main Usage Flow

1. Kerio Connect sends mail, security, and audit syslog to Logstash.
2. Logstash stores raw events in `kerio-connect-*` and aggregated mail-flow documents in `kerio-flow-*`.
3. This repository starts Grafana with pre-provisioned Elasticsearch data sources.
4. Grafana loads the `Kerio Mail Traffic` dashboard from `dashboards/`.
5. Engineers filter the dashboard by `run_id` from the mail load generator and inspect live traffic dynamics.

## Who This Is For

- Kerio Connect administrators who need a quick visual view of mail-test traffic.
- DevOps, observability, and lab engineers validating the Kerio syslog-to-Elasticsearch path.
- Project contributors who need a stable dashboard while changing the parser, generator, or verification workflow.

## Architecture / Component Roles

1. **Kerio Connect** emits operational mail logs through syslog.
2. **Logstash / Elasticsearch** are provided by `kerio-logstash-project` and create the data this dashboard reads.
3. **Grafana** is started by this repository through Docker Compose.
4. **Provisioning files** create the Elasticsearch data sources and load the dashboard without manual UI setup.
5. **Dashboard variables** map to mail load generator parameters such as `run_id`, `message_count`, `send_rate`, and `scenario_profile`.

## Requirements

### Software

- Linux host with Docker and Docker Compose.
- Network access from Grafana to Elasticsearch.
- Existing Elasticsearch indexes from `kerio-logstash-project`:
  - `kerio-flow-*`
  - `kerio-connect-*`

### Hardware

- CPU: 1 vCPU minimum for the Grafana-only stack.
- RAM: 512 MB minimum, 1 GB or more recommended.
- Disk: small persistent volume for Grafana state; dashboards and provisioning files are stored in the repository.

### Tested versions

| Component | Version | Notes |
|---|---:|---|
| Grafana OSS | 12.4.2 | Docker image pinned in `docker-compose.yml` |
| Elasticsearch | 8.19.11 | Tested with `kerio-logstash-project` indexes |
| Docker Compose | v5.1.1 | Tested on the Grafana VM |

## Repository structure

- `docker-compose.yml` starts Grafana and mounts provisioning files.
- `.env.example` documents required runtime variables.
- `provisioning/datasources/elasticsearch.yml` provisions `Kerio Mail Flows` and `Kerio Raw Events`.
- `provisioning/dashboards/kerio-mail.yml` tells Grafana where to load dashboards from.
- `dashboards/kerio-mail-traffic.json` is the provisioned dashboard.
- `README.md`, `README.ru.md`, `CHANGELOG.md`, `HANDOFF.md`, and `NEXT_STEPS.md` describe project state and next steps.
- `CONTRIBUTING.md`, `SECURITY.md`, `SUPPORT.md`, and `LICENSE` provide repository governance and publishing guidance.

## Documentation language policy

- `README.md` is the main English source.
- `README.ru.md` is the main Russian translation for lab work and quick onboarding.
- The first line of both README files is the language switcher:

```md
Language: [English](README.md) | [Русский](README.ru.md)
```

- The Russian README follows the English README and does not document separate behavior.
- If the English README changes, update `README.ru.md` in the same release when feasible. If not feasible, add a short translation freshness note near the top of `README.ru.md`.
- `CHANGELOG.md` is maintained in English. Do not create a Russian changelog unless the repository explicitly decides otherwise.
- `CONTRIBUTING.md` is maintained in English; Russian README changes are welcome when they preserve the meaning of the English version.

## Quick Start

Short path: configure Grafana, start the container, confirm the provisioned data sources, and open the Kerio mail dashboard.

- prepare `.env`;
- start Grafana;
- check Grafana health;
- check Elasticsearch data sources;
- open the dashboard and set the `run_id` variable.

### 1. Clone the repository

```bash
git clone https://github.com/foksk76/kerio-grafana-dashboard.git
cd kerio-grafana-dashboard
```

If all is well:

- the current directory is the repository root;
- `docker-compose.yml`, `.env.example`, and `dashboards/kerio-mail-traffic.json` are present.

### 2. Prepare the environment

```bash
cp .env.example .env
# edit .env for your lab hosts and passwords
```

**What you can edit**

- `GRAFANA_HTTP_PORT`
- `GRAFANA_ROOT_URL`
- `GRAFANA_ADMIN_USER`
- `GRAFANA_ADMIN_PASSWORD`
- `ELASTICSEARCH_URL`
- `ELASTICSEARCH_USER`
- `ELASTICSEARCH_PASSWORD`

**What matters**

- `ELASTICSEARCH_URL` must be reachable from the Grafana container.
- `ELASTICSEARCH_FLOW_INDEX` should usually stay `kerio-flow-*`.
- `ELASTICSEARCH_EVENTS_INDEX` should usually stay `kerio-connect-*`.
- `.env` contains secrets and is ignored by git.

### 3. Run the project

```bash
docker compose up -d
```

If all is well:

- `kerio-grafana` is running;
- Grafana listens on `${GRAFANA_HTTP_PORT:-3000}`;
- provisioning logs show dashboards and data sources loaded.

### 4. Verify the result

```bash
curl -sS http://127.0.0.1:3000/api/health
```

If all is well:

- the response contains `"database": "ok"`;
- the response includes the Grafana version.

```bash
set -a
. ./.env
set +a

curl -sS -u "$GRAFANA_ADMIN_USER:$GRAFANA_ADMIN_PASSWORD" \
  http://127.0.0.1:${GRAFANA_HTTP_PORT:-3000}/api/datasources/uid/kerio-flow-es/health

curl -sS -u "$GRAFANA_ADMIN_USER:$GRAFANA_ADMIN_PASSWORD" \
  http://127.0.0.1:${GRAFANA_HTTP_PORT:-3000}/api/datasources/uid/kerio-connect-es/health
```

If all is well:

- both responses contain `"status":"OK"`;
- the message says the Elasticsearch data source is healthy.

### 5. Confirm the outcome

```text
http://<grafana-host>:3000/d/kerio-mail-traffic/kerio-mail-traffic
```

Set the dashboard variables to match the mail load generator:

- `run_id`: value passed to `scripts/send_mail_batch.py --run-id`;
- `message_count`: value passed to `--message-count`;
- `send_rate`: value passed to `--send-rate`;
- `scenario_profile`: value passed to `--scenario-profile`.

Use `run_id=*` to show all test-run traffic with subjects matching `KT-*`.

After the steps above:

- Grafana is running with provisioned Elasticsearch data sources;
- the `Kerio Mail Traffic` dashboard is available;
- dashboard variables can be aligned with a known mail load run.

## Audit Matrix Run

`kerio-logstash-project/scripts/send_mail_batch.py` writes subjects in this form:

```text
KT-<run_id>-<sequence>-<scenario>
```

Dashboard panels filter that traffic with:

```text
email.subject.keyword:KT-${run_id:raw}-*
```

The `:raw` Grafana variable format is intentional. It lets `run_id=*` behave as a wildcard instead of a literal escaped asterisk.

This repository does not run a separate audit matrix by itself. Use the mail load generator from `kerio-logstash-project`, then filter Grafana by the generated `run_id`.

## Minimal Parser Event

Minimal matching flow document shape:

```json
{
  "@timestamp": "2026-04-08T12:03:40.000Z",
  "event": {
    "action": "message_flow_aggregated"
  },
  "email": {
    "subject": "KT-GRAFANA-DASHBOARD-20260408-120340-0001-basic",
    "from": {
      "address": "sender@example.test"
    },
    "to": {
      "address": ["recipient@example.test"]
    }
  },
  "kerio": {
    "mail": {
      "flow": {
        "result": "delivered"
      },
      "recipients": {
        "count": 1
      }
    }
  }
}
```

Minimal dashboard query:

```text
event.action:message_flow_aggregated AND email.subject.keyword:KT-<run_id>-*
```

## Normalized Result

When Elasticsearch contains matching data, the dashboard shows:

- top-line panels with numbers and history for key mail-flow indicators;
- time series for traffic dynamics and event volume;
- breakdowns by scenario, delivery result, and mail participants when those views are included in the current dashboard version.

## Verification checklist

- [ ] Repository cloned successfully
- [ ] `.env` created from `.env.example`
- [ ] Grafana container started
- [ ] Grafana health endpoint returned `database: ok`
- [ ] `kerio-flow-es` datasource returned `status: OK`
- [ ] `kerio-connect-es` datasource returned `status: OK`
- [ ] `Kerio Mail Traffic` dashboard opened
- [ ] Dashboard variables match a known mail load run

## Troubleshooting

### Problem: dashboard panels show `No data`

**Symptoms**

- Grafana opens, but charts are empty.

**What to check**

- time range does not include the mail load run;
- `run_id` does not match the subject prefix;
- Elasticsearch contains no `kerio-flow-*` or `kerio-connect-*` documents;
- datasource health is not `OK`.

**How to fix it**

```bash
set -a
. ./.env
set +a

curl -sS -u "$ELASTICSEARCH_USER:$ELASTICSEARCH_PASSWORD" \
  "$ELASTICSEARCH_URL/kerio-flow-*/_count"
```

Then set `run_id=*` and use a time range that includes the test run.

### Problem: Grafana datasource health fails

**Symptoms**

- datasource API returns `Failed to connect to Elasticsearch`;
- dashboard panels show datasource errors.

**What to check**

- `ELASTICSEARCH_URL` in `.env`;
- network path from the Grafana VM or container to Elasticsearch;
- Elasticsearch credentials;
- whether the ELK stack is running.

**How to fix it**

```bash
docker compose logs --tail=100 grafana
curl -sS -u "$ELASTICSEARCH_USER:$ELASTICSEARCH_PASSWORD" "$ELASTICSEARCH_URL/_cluster/health?pretty"
```

### Problem: wildcard `run_id=*` returns no data

**Symptoms**

- a concrete `run_id` works;
- `*` does not show all test traffic.

**What to check**

- dashboard queries should use `${run_id:raw}`, not `${run_id}`;
- the event subjects should start with `KT-`.

**How to fix it**

Update the dashboard JSON and reload provisioning:

```bash
docker compose restart grafana
```

## What This Project Does Not Do

- It does not deploy Kerio Connect.
- It does not deploy Logstash or Elasticsearch.
- It does not generate mail load by itself.
- It does not replace Grafana, Elasticsearch, or Kerio vendor documentation.
- It is not a production hardening guide.

## What To Know Before Use

- `.env` contains secrets and should not be committed.
- Grafana state is stored in the named Docker volume `grafana_data`.
- The dashboard expects ECS-like fields produced by `kerio-logstash-project`.
- Single-node Elasticsearch may report `yellow` cluster health if indexes request replicas.
- Vendor licenses and terms for Kerio Connect, Grafana, Elasticsearch, and Docker images remain separate from this repository.

## Roadmap

See [NEXT_STEPS.md](./NEXT_STEPS.md)

## Changelog

See [CHANGELOG.md](./CHANGELOG.md)

Keep `CHANGELOG.md` canonical and English-only unless this repository explicitly decides otherwise.

## Handoff

See [HANDOFF.md](./HANDOFF.md)

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md)

## GitHub Release Notes

GitHub Release Notes stay in English.

Recommended sections:

```md
## Operational Changes

- What users can now run, observe, validate, or troubleshoot.

## Validation

- Exact local checks, live run IDs, CI run status, and expected pass/fail numbers.

## Engineer Notes

- Required engineer action, configuration change, migration note, known limitation, manual-only step, or superseded release note.
```

## Security

See [SECURITY.md](./SECURITY.md)

## Support

See [SUPPORT.md](./SUPPORT.md)

## License

This project is licensed under the [Apache License 2.0](./LICENSE), matching the adjacent `kerio-logstash-project` repository.
