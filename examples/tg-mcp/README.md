# examples/tg-mcp — деплой-комплект (обезличено)

Боевые конфиги для `fast-mcp-telegram`, вычищенные до шаблона. Подставь свой домен,
секрет и токен. Полный путь с объяснениями — в [`../../02_TELEGRAM_MCP.md`](../../02_TELEGRAM_MCP.md).

## Файлы

| Файл | Что | Что подставить |
|---|---|---|
| `docker-compose.yml` | контейнер, собирается из PyPI (образ автора закрыт) | `DOMAIN` |
| `.env.example` | переменные контейнера | `API_ID`, `API_HASH`, `DOMAIN` |
| `nginx.conf.example` | реверс-прокси + секретный путь для мобилы | домен, `<SECRET>`, `<TOKEN>` |

## Быстрый старт

```bash
mkdir -p /root/tg-mcp/sessions && cd /root/tg-mcp
# скопируй сюда docker-compose.yml, nginx.conf.example, .env.example
cp .env.example .env && chmod 600 .env      # впиши API_ID / API_HASH / DOMAIN
chown -R 1000:1000 sessions/                # ← контейнер под uid=1000 (иначе /setup не пишет сессию)
docker compose up -d --build                # 127.0.0.1:8810, эндпойнт /v1/mcp
docker compose ps                           # tg-mcp — Up (healthy)

# nginx: домен + <SECRET> (openssl rand -hex 16)
cp nginx.conf.example /etc/nginx/sites-available/tg-mcp
ln -s /etc/nginx/sites-available/tg-mcp /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
certbot --nginx -d tg-mcp.example.com

# логин: https://tg-mcp.example.com/setup — ТОЛЬКО по номеру (QR не умеет 2FA) → Bearer-токен
# впиши <TOKEN> в мобильную локацию nginx, закрой /setup (allow <YOUR_IP>; deny all;)
```

## Три вещи, на которых теряют полчаса

1. `chown -R 1000:1000 sessions/` — контейнер под uid=1000, иначе `/setup` не запишет сессию.
2. Эндпойнт — **`/v1/mcp`**, не `/mcp`.
3. Порт `8810` — только на `127.0.0.1`, наружу один nginx. `/setup` закрой allow/deny по IP.

Грабли целиком — [`../../02_TELEGRAM_MCP.md §6`](../../02_TELEGRAM_MCP.md). Безопасность —
[`../../01_SECURITY.md`](../../01_SECURITY.md).
