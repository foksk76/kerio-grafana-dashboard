# Contributing

Thank you for your interest in contributing.

## Ways to contribute

You can help by:

- reporting dashboard or provisioning bugs;
- improving documentation;
- suggesting new panels or dashboard variables;
- improving validation commands;
- fixing reproducibility issues.

## Before opening a pull request

Please:

1. read the current `README.md`;
2. verify your change locally;
3. update documentation when behavior changes;
4. keep examples realistic and safe to publish;
5. avoid unrelated cleanup in the same pull request.

## Documentation expectations

Documentation in this project family should be:

- written primarily in English;
- practical and copy-paste friendly;
- reproducible without hidden assumptions;
- explicit about expected results and verification steps.

When changing documentation, please include:

- exact commands where possible;
- example input and output when relevant;
- a short verification method;
- notes about limitations or assumptions.

Russian README updates are welcome when they preserve the meaning of the English README and do not document separate behavior.

## Dashboard and configuration changes

For dashboard, Compose, or provisioning updates, please include:

- a short explanation of the operational change;
- expected behavior;
- validation commands or API checks;
- compatibility notes if Grafana or Elasticsearch behavior is affected.

Recommended local checks:

```bash
docker compose config
python3 -m json.tool dashboards/kerio-mail-traffic.json >/tmp/kerio-mail-traffic.json.check
```

## Issues

When reporting a problem, please include:

- repository name;
- Grafana, Elasticsearch, and Docker Compose versions;
- the exact dashboard variable values;
- the exact time range;
- actual result;
- expected result;
- relevant logs or screenshots if safe to share.

## Sensitive data

Do not include:

- real credentials;
- private email data;
- non-anonymized production logs;
- proprietary binaries or artifacts that cannot be redistributed.

## Pull request style

Please keep pull requests:

- focused;
- easy to review;
- documented;
- consistent with the repository's existing file naming and structure.
