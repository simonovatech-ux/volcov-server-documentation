# Эксплуатация и диагностика

Все команды выполняются от имени администратора сервера. Перед изменениями необходимо сделать резервную копию и проверить свободное место.

## Быстрая проверка

```bash
uptime
free -h
df -hT -x tmpfs -x devtmpfs
systemctl --failed --no-pager
docker compose ls --all
docker ps -a
```

Публичные проверки:

```bash
curl -fsS https://volcov-ai-mcp.duckdns.org/health
curl -fsS https://volcov-ai-mcp.duckdns.org/frs-health
curl -fsS https://volcov-hr-bot.duckdns.org/health
```

Ожидаемое состояние основного и FRS endpoint — HTTP `200`. HR endpoint должен вернуть `200` после восстановления `hiring-app`; на момент инвентаризации он возвращал `502`.

## Работа с Compose-проектом

Пример для чтения состояния без изменений:

```bash
cd /opt/demo-flow-mcp
docker compose ps
docker compose logs --tail=200
docker compose config --services
```

Перезапуск или пересборку следует выполнять отдельно для конкретного проекта и только после проверки `.env` и резервной копии:

```bash
cd /opt/PROJECT
docker compose up -d --build
docker compose ps
```

## Диагностика `hiring-app`

До запуска контейнера необходимо сохранить журналы остановленного экземпляра:

```bash
docker inspect hiring-app
docker logs --timestamps --tail=500 hiring-app
cd /opt/hiring
docker compose config --services
docker compose ps -a
```

Код `137` означает получение `SIGKILL`. `OOMKilled=false` не подтверждает системный OOM, поэтому нужно проверить одновременно:

- системный журнал за время остановки;
- ручные операции или деплой 4 сентября;
- потребление памяти;
- события Docker;
- доступность обязательных переменных окружения и зависимостей.

## Журналы

```bash
journalctl -u docker --since '24 hours ago' --no-pager
journalctl -u ssh --since '24 hours ago' --no-pager
docker logs --tail=200 CONTAINER
```

Перед публикацией журналов нужно удалять токены, адреса webhook, персональные данные кандидатов и содержимое сообщений.

## Резервные копии

На сервере обнаружены:

- `/opt/volcov-offsite-backups/20260831T065102Z`;
- `/opt/hiring-backups` с дампами PostgreSQL и архивами кода;
- `/opt/demo-flow-backups`;
- `/root/create_volcov_server_backup.sh`;
- несколько rollback-каталогов приложений.

Расписание пользовательского `crontab` не настроено. Наличие скрипта само по себе не гарантирует регулярный запуск. Нужно отдельно задокументировать целевое хранилище, срок хранения, шифрование и тест восстановления.

## Рекомендуемый регламент

- ежедневно: HTTP health-check и проверка контейнеров;
- еженедельно: проверка свободного места, обновлений и размера журналов;
- перед каждым деплоем: дамп базы и архив текущего кода;
- ежемесячно: тест восстановления одной базы и одного приложения;
- ежеквартально: ротация административных ключей и ревизия доступов.

