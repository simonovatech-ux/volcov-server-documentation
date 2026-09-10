# Архитектура

## Общая схема

```mermaid
flowchart TD
    Internet[Интернет] --> Caddy[Caddy :80/:443]
    Admin[Администратор] --> SSH[OpenSSH :22]

    Caddy --> MainMCP[demo-flow-mcp / MCP :3000]
    Caddy --> FRS[frs-statosphera-mcp :3000]
    Caddy --> HiringApp[hiring-app]

    MainMCP --> Transcriber[transcriber :8080]
    MainMCP --> Auditor[ai-usage-auditor :8000]
    MainMCP --> SharedData[/opt/demo-flow-mcp/data]

    HiringApp --> HiringDB[(PostgreSQL 16)]
    Novinki[novinki-bot :8092 localhost] --> NovinkiDB[(PostgreSQL 17)]

    Bitrix[bitrix-plan-autofill] --> BitrixData[/opt/bitrix-plan-autofill/data]
```

На момент инвентаризации ветка `Caddy → hiring-app` не работает: контейнер приложения остановлен, поэтому внешний HR health-check возвращает `502`.

## Docker-сети

### `demo-flow-mcp_internal`

Общая внутренняя сеть интеграционных сервисов:

- `demo-flow-mcp-mcp-1`;
- `demo-flow-mcp-transcriber-1`;
- `demo-flow-mcp-caddy-1`;
- `frs-statosphera-mcp-frs-mcp-1`;
- `ai-usage-auditor`;
- `novinki-bot`;
- остановленные `hiring-app` и `sber-max-bot-hiring-bot-1`.

### `hiring_internal`

- `hiring-db` — PostgreSQL 16;
- `hiring-app` — приложение найма, сейчас остановлено;
- `volcov-hiring-db-console` — pgweb, доступен только на `127.0.0.1:8088`.

### `app_database`

- `app-postgres-1` — PostgreSQL 17;
- `novinki-bot`;
- `volcov-novinki-db-console`.

### Отдельные сети

`bitrix-plan-autofill` использует собственную Compose-сеть. Остановленный `hr-bot-tunnel` использовал сеть хоста.

## Хранилища

| Назначение | Тип | Расположение |
|---|---|---|
| Данные основного MCP | Bind mount | `/opt/demo-flow-mcp/data` |
| Задания транскрибации | Docker volume | `demo-flow-mcp_whisperx_jobs` |
| Модели транскрибации | Docker volume | `demo-flow-mcp_whisperx_models` |
| Данные найма | Bind mount | `/opt/hiring/data` |
| База найма | Docker volume | `hiring_pgdata` |
| Данные «Новинок» | Bind mounts | `/opt/novinki/app/data`, `/opt/novinki/catalog` |
| База «Новинок» | Docker volume | `app_postgres_data` |
| Данные планов Битрикс | Bind mount | `/opt/bitrix-plan-autofill/data` |
| Сертификаты Caddy | Docker volumes | `demo-flow-mcp_caddy_data`, `demo-flow-mcp_caddy_config` |

PostgreSQL не опубликован напрямую в интернет. Это правильное ограничение: доступ к базам должен идти через контейнерную сеть или SSH-туннель.

