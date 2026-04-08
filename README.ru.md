Язык: [English](README.md) | [Русский](README.ru.md)

# kerio-grafana-dashboard

Автонастройка Grafana для почтового трафика Kerio Connect, сохраненного в Elasticsearch.

> **Статус проекта:** лабораторный проект для безопасной проверки визуализации Kerio Connect mail-flow перед боевым использованием.

> **Языковая политика:** `README.md` - основной README на английском языке. `README.ru.md` - основной русский перевод для лабораторной работы и быстрого знакомства с проектом. Сохраняйте переключатель языка первой строкой в обоих файлах.

## Зачем нужен этот репозиторий

Почтовые журналы Kerio Connect проще сопровождать, когда распарсенные события и агрегированные Queue-ID потоки видны как живые графики.

Цель этой лаборатории - измерить и отобразить показатели почтового потока для мониторинга, выявления отклонений и последующего переноса подхода на боевую среду.

Этот репозиторий добавляет небольшой воспроизводимый слой Grafana для лаборатории Kerio Connect logging. Он автоматически создает в Grafana источники данных Elasticsearch и дашборд почтового трафика. Дашборд помогает видеть динамику почтового прогона и сопоставлять ее с параметрами, которые задаются в скрипте генерации нагрузки.

## Семейство проектов

Этот репозиторий является частью семейства проектов **Kerio Connect Monitoring & Logging**:

1. [kerio-connect](https://github.com/foksk76/kerio-connect) - воспроизводимое лабораторное окружение Kerio Connect
2. [kerio-logstash-project](https://github.com/foksk76/kerio-logstash-project) - пайплайн парсинга, нормализации и проверки Kerio syslog для ELK
3. [kerio-syslog-anonymizer](https://github.com/foksk76/kerio-syslog-anonymizer) - детерминированная анонимизация реальных логов для безопасной публикации и повторяемой корреляции
4. `kerio-grafana-dashboard` - слой автонастройки Grafana и дашборда для почтового трафика Kerio в Elasticsearch

## Место репозитория в общей схеме

```text
Kerio Connect -> Syslog (RFC5424) -> Logstash -> Elasticsearch -> Grafana
```

- `kerio-connect` поднимает лабораторный Kerio Connect host.
- `kerio-logstash-project` парсит Kerio syslog и пишет индексы `kerio-connect-*` и `kerio-flow-*`.
- `kerio-syslog-anonymizer` готовит реальные логи к безопасной публикации, когда это нужно.
- Этот репозиторий предоставляет конфигурацию для запуска Grafana и дашборд почтового трафика.

## Основной сценарий использования

1. Инженер запускает генерацию синтетического почтового трафика по заданным параметрам: `run_id`, количеству сообщений, скорости отправки и профилю сценариев.
2. Kerio Connect обрабатывает этот трафик и отправляет почтовые, security и audit logs в Logstash через syslog.
3. Logstash сохраняет исходные события в `kerio-connect-*` и агрегированные mail-flow documents в `kerio-flow-*`.
4. Этот репозиторий предоставляет конфигурацию Grafana с заранее настроенными источниками данных Elasticsearch.
5. Grafana загружает дашборд `Kerio Mail Traffic` из `dashboards/`.
6. Инженер фильтрует дашборд по `run_id` из генератора почтовой нагрузки и смотрит динамику трафика.

## Для кого это

- Администраторы Kerio Connect, которым нужен быстрый визуальный обзор тестового почтового трафика.
- DevOps, observability и lab-инженеры, которые проверяют путь Kerio syslog -> Elasticsearch.
- Участники проекта, которым нужен информативный дашборд при изменениях в парсере, генераторе или процессе верификации.

## Архитектура / Роли компонентов

1. **Kerio Connect** отправляет почтовые операционные журналы через syslog.
2. **Logstash / Elasticsearch** предоставляются `kerio-logstash-project` и создают данные для дашборда.
3. **Grafana** запускается через Docker Compose на основе конфигурации из этого репозитория.
4. **Файлы автонастройки** создают в Grafana источники данных Elasticsearch и загружают дашборд без ручной настройки UI.
5. **Dashboard variables** соответствуют параметрам генератора нагрузки: `run_id`, `message_count`, `send_rate`, `scenario_profile`.

## Требования

### Программное обеспечение

- Linux host с Docker и Docker Compose.
- Сетевой доступ от Grafana к Elasticsearch.
- Существующие Elasticsearch индексы из `kerio-logstash-project`:
  - `kerio-flow-*`
  - `kerio-connect-*`

### Аппаратные ресурсы

- CPU: минимум 1 vCPU для Grafana-only стека.
- RAM: минимум 512 MB, рекомендуется 1 GB или больше.
- Диск: небольшой persistent volume для состояния Grafana; dashboards и файлы автонастройки лежат в репозитории.

### Проверенные версии

| Компонент | Версия | Примечания |
|---|---:|---|
| Grafana OSS | 12.4.2 | Docker image закреплен в `docker-compose.yml` |
| Elasticsearch | 8.19.11 | Проверено с индексами `kerio-logstash-project` |
| Docker Compose | v5.1.1 | Проверено на Grafana VM |

## Структура репозитория

- `docker-compose.yml` запускает Grafana и монтирует файлы автонастройки.
- `.env.example` документирует обязательные runtime variables.
- `provisioning/datasources/elasticsearch.yml` описывает источники данных `Kerio Mail Flows` и `Kerio Raw Events`.
- `provisioning/dashboards/kerio-mail.yml` указывает Grafana, откуда загружать dashboards.
- `dashboards/kerio-mail-traffic.json` - автоматически загружаемый дашборд.
- `README.md`, `README.ru.md`, `CHANGELOG.md`, `HANDOFF.md` и `NEXT_STEPS.md` описывают состояние проекта и следующие шаги.
- `CONTRIBUTING.md`, `SECURITY.md`, `SUPPORT.md` и `LICENSE` задают правила сопровождения и публикации репозитория.

## Языковая политика документации

- `README.md` - основной источник на английском языке.
- `README.ru.md` - основной русский перевод для лабораторной работы и быстрого знакомства с проектом.
- Первая строка обоих README-файлов - переключатель языка:

```md
Language: [English](README.md) | [Русский](README.ru.md)
```

- Русский README следует английскому README и не описывает отдельное поведение.
- Если английский README меняется, обновляйте `README.ru.md` в том же релизе, когда это возможно. Если это невозможно, добавьте короткую заметку об актуальности перевода в начало `README.ru.md`.
- `CHANGELOG.md` ведется на английском языке. Не создавайте русский changelog, если репозиторий явно не решил иначе.
- `CONTRIBUTING.md` ведется на английском языке; русские правки README приветствуются, если они сохраняют смысл английской версии.

## Быстрый старт

Короткий путь: настроить Grafana, запустить контейнер, проверить автоматически настроенные источники данных и открыть Kerio mail dashboard.

- подготовить `.env`;
- запустить Grafana;
- проверить Grafana health;
- проверить источники данных Elasticsearch;
- открыть dashboard и задать переменную `run_id`.

### 1. Клонируйте репозиторий

```bash
git clone https://github.com/foksk76/kerio-grafana-dashboard.git
cd kerio-grafana-dashboard
```

Если все хорошо:

- текущая директория является корнем репозитория;
- присутствуют `docker-compose.yml`, `.env.example` и `dashboards/kerio-mail-traffic.json`.

### 2. Подготовьте окружение

```bash
cp .env.example .env
# отредактируйте .env под свои lab hosts и пароли
```

**Что можно изменить**

- `GRAFANA_HTTP_PORT`
- `GRAFANA_ROOT_URL`
- `GRAFANA_ADMIN_USER`
- `GRAFANA_ADMIN_PASSWORD`
- `ELASTICSEARCH_URL`
- `ELASTICSEARCH_USER`
- `ELASTICSEARCH_PASSWORD`

**Что важно**

- `ELASTICSEARCH_URL` должен быть доступен из Grafana container.
- `ELASTICSEARCH_FLOW_INDEX` обычно остается `kerio-flow-*`.
- `ELASTICSEARCH_EVENTS_INDEX` обычно остается `kerio-connect-*`.
- `.env` содержит секреты и игнорируется git.

### 3. Запустите проект

```bash
docker compose up -d
```

Если все хорошо:

- `kerio-grafana` запущен;
- Grafana слушает `${GRAFANA_HTTP_PORT:-3000}`;
- в логах автонастройки видно, что dashboards и источники данных загружены.

### 4. Проверьте результат

```bash
curl -sS http://127.0.0.1:3000/api/health
```

Если все хорошо:

- ответ содержит `"database": "ok"`;
- ответ содержит версию Grafana.

```bash
set -a
. ./.env
set +a

curl -sS -u "$GRAFANA_ADMIN_USER:$GRAFANA_ADMIN_PASSWORD" \
  http://127.0.0.1:${GRAFANA_HTTP_PORT:-3000}/api/datasources/uid/kerio-flow-es/health

curl -sS -u "$GRAFANA_ADMIN_USER:$GRAFANA_ADMIN_PASSWORD" \
  http://127.0.0.1:${GRAFANA_HTTP_PORT:-3000}/api/datasources/uid/kerio-connect-es/health
```

Если все хорошо:

- оба ответа содержат `"status":"OK"`;
- сообщение говорит, что источник данных Elasticsearch исправен.

### 5. Зафиксируйте итог

```text
http://<grafana-host>:3000/d/kerio-mail-traffic/kerio-mail-traffic
```

Задайте dashboard variables в соответствии с генератором почтовой нагрузки:

- `run_id`: значение `scripts/send_mail_batch.py --run-id`;
- `message_count`: значение `--message-count`;
- `send_rate`: значение `--send-rate`;
- `scenario_profile`: значение `--scenario-profile`.

Используйте `run_id=*`, чтобы показать весь тестовый трафик с subject по шаблону `KT-*`.

После шагов выше:

- Grafana запущена с автоматически настроенными источниками данных Elasticsearch;
- dashboard `Kerio Mail Traffic` доступен;
- dashboard variables можно сопоставить с известным mail load run.

## Проверка Audit Matrix

`kerio-logstash-project/scripts/send_mail_batch.py` пишет subject в таком виде:

```text
KT-<run_id>-<sequence>-<scenario>
```

Панели фильтруют этот трафик так:

```text
email.subject.keyword:KT-${run_id:raw}-*
```

Формат Grafana variable `:raw` используется намеренно. Он позволяет `run_id=*` работать как wildcard, а не как literal escaped asterisk.

В этом репозитории нет отдельного запуска audit matrix. Используйте генератор почтовой нагрузки из `kerio-logstash-project`, затем фильтруйте Grafana по созданному `run_id`.

## Минимальный пример события

Минимальная форма подходящего flow document:

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

Минимальный dashboard query:

```text
event.action:message_flow_aggregated AND email.subject.keyword:KT-<run_id>-*
```

## Нормализованный результат

Когда в Elasticsearch есть подходящие данные, dashboard показывает:

- верхние панели с числами и историей для ключевых показателей почтового потока;
- временные ряды для динамики трафика и объема событий;
- разрезы по сценариям, результатам доставки и участникам почтового обмена, если они включены в текущую версию дашборда.

## Чеклист проверки

- [ ] Репозиторий успешно клонирован
- [ ] `.env` создан из `.env.example`
- [ ] Grafana container запущен
- [ ] Grafana health endpoint вернул `database: ok`
- [ ] `kerio-flow-es` datasource вернул `status: OK`
- [ ] `kerio-connect-es` datasource вернул `status: OK`
- [ ] Dashboard `Kerio Mail Traffic` открылся
- [ ] Dashboard variables соответствуют известному mail load run

## Устранение неполадок

### Проблема: dashboard panels показывают `No data`

**Симптомы**

- Grafana открывается, но графики пустые.

**Что проверить**

- time range не включает mail load run;
- `run_id` не совпадает с subject prefix;
- в Elasticsearch нет документов `kerio-flow-*` или `kerio-connect-*`;
- datasource health не `OK`.

**Как исправить**

```bash
set -a
. ./.env
set +a

curl -sS -u "$ELASTICSEARCH_USER:$ELASTICSEARCH_PASSWORD" \
  "$ELASTICSEARCH_URL/kerio-flow-*/_count"
```

Затем задайте `run_id=*` и выберите time range, который включает тестовый прогон.

### Проблема: Grafana datasource health падает

**Симптомы**

- datasource API возвращает `Failed to connect to Elasticsearch`;
- dashboard panels показывают datasource errors.

**Что проверить**

- `ELASTICSEARCH_URL` в `.env`;
- сетевой путь от Grafana VM или container до Elasticsearch;
- Elasticsearch credentials;
- запущен ли ELK stack.

**Как исправить**

```bash
docker compose logs --tail=100 grafana
curl -sS -u "$ELASTICSEARCH_USER:$ELASTICSEARCH_PASSWORD" "$ELASTICSEARCH_URL/_cluster/health?pretty"
```

### Проблема: wildcard `run_id=*` не возвращает данные

**Симптомы**

- конкретный `run_id` работает;
- `*` не показывает весь тестовый трафик.

**Что проверить**

- dashboard queries должны использовать `${run_id:raw}`, а не `${run_id}`;
- subject события должен начинаться с `KT-`.

**Как исправить**

Обновите dashboard JSON и перезагрузите автонастройку:

```bash
docker compose restart grafana
```

## Что проект не делает

- Не разворачивает Kerio Connect.
- Не разворачивает Logstash или Elasticsearch.
- Не генерирует почтовую нагрузку самостоятельно.
- Не заменяет документацию Grafana, Elasticsearch или Kerio.
- Не является руководством по production hardening.

## Что важно знать

- `.env` содержит секреты и не должен попадать в git.
- Состояние Grafana хранится в named Docker volume `grafana_data`.
- Dashboard ожидает ECS-like поля, создаваемые `kerio-logstash-project`.
- Single-node Elasticsearch может показывать `yellow` cluster health, если индексы требуют replicas.
- Лицензии и условия Kerio Connect, Grafana, Elasticsearch и Docker images остаются отдельными от этого репозитория.

## Roadmap

См. [NEXT_STEPS.md](./NEXT_STEPS.md)

## Changelog

См. [CHANGELOG.md](./CHANGELOG.md)

Оставляйте `CHANGELOG.md` каноническим и англоязычным, если этот репозиторий явно не решил иначе.

## Handoff

См. [HANDOFF.md](./HANDOFF.md)

## Участие в проекте

См. [CONTRIBUTING.md](./CONTRIBUTING.md)

## GitHub Release Notes

GitHub Release Notes остаются на английском языке.

Рекомендуемые разделы:

```md
## Operational Changes

- What users can now run, observe, validate, or troubleshoot.

## Validation

- Exact local checks, live run IDs, CI run status, and expected pass/fail numbers.

## Engineer Notes

- Required engineer action, configuration change, migration note, known limitation, manual-only step, or superseded release note.
```

## Безопасность

См. [SECURITY.md](./SECURITY.md)

## Поддержка

См. [SUPPORT.md](./SUPPORT.md)

## Лицензия

Проект опубликован под [Apache License 2.0](./LICENSE), как и соседний репозиторий `kerio-logstash-project`.
