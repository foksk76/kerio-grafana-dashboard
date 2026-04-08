# Next Steps

Updated: 2026-04-08

## Immediate Steps

1. Add a CI workflow after the repository is initialized on GitHub.
2. Add screenshot-based dashboard review only if screenshots can avoid secrets and private traffic data.
3. Decide whether the dashboard should keep the current compact layout or support a second wide-screen variant.
4. Consider adding release screenshots or exported dashboard previews after sanitizing any lab-specific values.

## Suggested CI Checks

- `docker compose config`
- `python3 -m json.tool dashboards/kerio-mail-traffic.json`
- no committed `.env`
- README language switcher present in both README files
- required governance files present

## Dashboard Follow-Ups

- Add datasource query smoke tests using Grafana `/api/ds/query`.
- Add a short dashboard validation script that checks panel IDs, data source UIDs, and `${run_id:raw}` usage.
- Consider a panel for Logstash unparsed tags if parser regression work becomes frequent.

## Documentation Follow-Ups

- Keep `README.ru.md` aligned with `README.md` when workflow or dashboard behavior changes.
- Keep `CHANGELOG.md` canonical and English-only.
- Update `HANDOFF.md` after live validation changes materially.
