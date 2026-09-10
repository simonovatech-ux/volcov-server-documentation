# Реестр сервисов

Состояние зафиксировано 10 сентября 2026 года.

## Compose-проекты

### `demo-flow-mcp`

- Путь: `/opt/demo-flow-mcp`.
- Сервисы: `mcp`, `transcriber`, `caddy`.
- Основной публичный домен: `volcov-ai-mcp.duckdns.org`.
- Проверка `/health`: HTTP `200`.
- Транскрибер имеет Docker health-check и состояние `healthy`.
- Используются Node.js, MCP SDK, Express, JOSE, Sharp и Zod.
- Каталог содержит код интеграции с Битрикс24, OAuth/DCR, рабочие процессы демонстраций, публичные ассеты и локальную транскрибацию.

### `frs-statosphera-mcp`

- Путь: `/opt/frs-statosphera-mcp`.
- Сервис: `frs-mcp`.
- Публичная проверка `/frs-health`: HTTP `200`.
- Назначение: MCP-адаптер для операций со Статосферой.
- Технологии: Node.js, MCP SDK, Express, Zod.
- Docker health-check не настроен; доступность сейчас контролируется внешним HTTP-запросом.

### `hiring`

- Путь: `/opt/hiring`.
- Сервисы: `db`, `app`.
- `hiring-db`: работает, PostgreSQL 16, health `healthy`.
- `hiring-app`: остановлен с кодом `137`; Docker не пометил остановку как OOM.
- Публичный домен: `volcov-hr-bot.duckdns.org`.
- Проверка `/health`: HTTP `502`.
- Назначение: обработка кандидатов, интеграции MAX, почты, HeadHunter/СберПодбора и недельная отчётность.
- В коде используются PostgreSQL, IMAP, обработка писем, Excel и LLM SDK.

### `novinki`

- Путь приложения: `/opt/novinki/app`.
- Сервисы: `novinki`, PostgreSQL 17.
- `novinki-bot` работает; локальный порт `127.0.0.1:8092`.
- `app-postgres-1` работает и имеет состояние `healthy`.
- Назначение: MAX-помощник продавца по новинкам, каталог и публикация карточек.
- Дополнительно запущен контейнер pgweb для администрирования базы.

### `ai-usage-auditor`

- Путь: `/opt/ai-usage-auditor`.
- Сервис работает, health `healthy`.
- Назначение: аудит использования ChatGPT/ИИ и MCP-доступ к результатам аудита.
- Подключён к общей внутренней сети `demo-flow-mcp_internal`.

### `bitrix-plan-autofill`

- Путь: `/opt/bitrix-plan-autofill`.
- Python-приложение работает без Docker health-check.
- Назначение: автоматическое создание или обновление файлов планов в Битрикс24 на основе шаблона Excel.
- Постоянные файлы находятся в `/opt/bitrix-plan-autofill/data`.

### `sber-max-bot`

- Путь: `/opt/sber-max-bot`.
- Контейнер остановлен с кодом `137`.
- Связанный `hr-bot-tunnel` также остановлен штатно с кодом `0`.
- Назначение: пилот решений руководителя и передача событий СберПодбора в MAX.
- Перед удалением необходимо подтвердить, заменён ли этот контур приложением `/opt/hiring`.

## Инфраструктурные контейнеры

| Контейнер | Состояние | Назначение |
|---|---|---|
| `demo-flow-mcp-caddy-1` | Работает | HTTPS, сертификаты, маршрутизация |
| `hiring-db` | Healthy | PostgreSQL 16 для найма |
| `app-postgres-1` | Healthy | PostgreSQL 17 для «Новинок» |
| `volcov-hiring-db-console` | Работает | pgweb для базы найма, только localhost |
| `volcov-novinki-db-console` | Работает | pgweb для базы «Новинок» |

## Системные службы

Работают Docker, containerd, OpenSSH, Fail2ban, cron, rsyslog, NetworkManager, systemd-networkd/resolved и unattended-upgrades. Упавших systemd-служб при проверке не обнаружено.

