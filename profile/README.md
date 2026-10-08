# MGate

**MGate** — почтовый сервер для собственной инфраструктуры. В один стек Docker Compose (или Helm-чарт)
собраны Postfix, Dovecot, Rspamd, ClamAV, DNSSEC-резолвер Unbound, Traefik (по умолчанию
самоподписанный сертификат, собственный сертификат загружается в админке; автоматический выпуск
Let's Encrypt — в разработке), а также веб-почта, админ-панель и личный кабинет пользователя.

- **Репозиторий:** [mgate-server/mgate](https://github.com/mgate-server/mgate) — комплект развёртывания,
  конфигурации почтовых сервисов и исходный код веб-интерфейса.
- **Релизы и список изменений:** [github.com/mgate-server/mgate/releases](https://github.com/mgate-server/mgate/releases)
- **Образы контейнеров:** [ghcr.io/mgate-server](https://github.com/orgs/mgate-server/packages) — публичные,
  `docker pull` работает без авторизации.

## Быстрый старт

```sh
git clone https://github.com/mgate-server/mgate.git && cd mgate   # ветка main = последний релиз
make init                        # создаёт .env и генерирует секреты
$EDITOR .env                     # MGATE_HOSTNAME, MGATE_ADMIN_EMAIL
docker compose pull && docker compose up -d
```

Если `MGATE_ADMIN_PASSWORD` в `.env` пуст, пароль первого администратора генерируется при первом запуске
и один раз выводится в лог: `docker compose logs migrate | grep -A2 'initial superadmin'`. Забытый пароль
сбрасывается командой `docker compose exec api mgate admin reset-password <email>`.

По умолчанию отдаётся самоподписанный сертификат: для рабочей эксплуатации загрузите собственный сертификат
в админке. Двухфакторная аутентификация (TOTP) пока в разработке — вход выполняется только по паролю.

Подробности — в [README](https://github.com/mgate-server/mgate#readme) и
[руководстве по развёртыванию](https://github.com/mgate-server/mgate/blob/main/docs/DEPLOYMENT.md).

## Образы

| Образ | Назначение |
|---|---|
| `ghcr.io/mgate-server/mgate` | backend: REST API, шлюз веб-почты, фоновые задачи, миграции |
| `ghcr.io/mgate-server/mgate-agent` | агент управления почтовыми контейнерами |
| `ghcr.io/mgate-server/web` | веб-интерфейс: админ-панель, веб-почта, личный кабинет |
| `ghcr.io/mgate-server/postfix` | MTA (SMTP, submission) |
| `ghcr.io/mgate-server/dovecot` | IMAP, POP3, LMTP, Sieve |
| `ghcr.io/mgate-server/rspamd` | антиспам, DKIM/ARC-подпись |
| `ghcr.io/mgate-server/unbound` | DNSSEC-валидирующий рекурсивный резолвер |
| `ghcr.io/mgate-server/traefik` | HTTPS-прокси |
| `ghcr.io/mgate-server/backup` | резервное копирование (restic + pg_dump) |
| `ghcr.io/mgate-server/firewall` | опциональная блокировка IP через nftables |

Версии выпускаются по CalVer `ГГГГ.М.N` (например, `2026.10.1`). У каждого образа есть неизменяемый
тег версии, а также плавающие теги `ГГГГ.М` (последняя сборка месяца) и `latest`. В продакшене
закрепляйте точную версию (`MGATE_VERSION` в `.env`).

## Безопасность

Об уязвимостях сообщайте приватно:
[Report a vulnerability](https://github.com/mgate-server/mgate/security/advisories/new).
