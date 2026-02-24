# Telegram Bot Starter on n8n

Стартовый проект для разработки Telegram-ботов на `n8n` с запуском в Docker и разработкой через AI-агентов в Cursor (`n8n-mcp`).

## 1) Переменные окружения

Скопируйте пример и при необходимости отредактируйте:

```bash
cp .env.example .env
```

В `.env` задайте:
- `N8N_API_KEY` — ключ из n8n (Settings → n8n API), нужен для MCP.
- `WEBHOOK_URL` — публичный URL для вебхуков (см. раздел про ngrok ниже). Без него Telegram не сможет доставлять сообщения на локальный n8n.

## 2) Поднять n8n локально

```bash
docker compose up -d
```

Проверка:

```bash
docker compose ps
docker compose logs -f n8n
```

UI: `http://localhost:5678`

## 3) Первый вход в n8n

1. Откройте `http://localhost:5678`.
2. Создайте owner-аккаунт (первичный onboarding n8n).
3. Перейдите в `Settings` -> `n8n API`.
4. Создайте API key.

## 4) Настроить AI-агента (Cursor + n8n-mcp)

Файл: `.cursor/mcp.json`

Убедитесь, что в нем заданы:
- `command: "npx"`
- `args: ["n8n-mcp"]`
- `MCP_MODE=stdio`
- `N8N_API_URL=http://localhost:5678`
- `N8N_API_KEY=<ваш ключ из n8n>`

После этого включите MCP server `n8n-mcp` в Cursor.

## 5) Smoke-test через AI-агента

Минимальный тест:

1. В чате Cursor попросите агента проверить доступность n8n через MCP.
2. Попросите получить список workflow (или документацию нод).
3. Если список/документация возвращаются без ошибок — связка `n8n + n8n-mcp` работает.

## 6) Где хранить автоматизации

Все ваши workflow хранятся в папке `workflows/`.

- Пример: `workflows/telegram-echo-workflow.json`

Правило: каждый workflow JSON должен иметь поле `description`.

## 7) ngrok — чтобы бот достучался до локального n8n

Telegram шлёт события на вебхук по публичному URL. Для локальной разработки нужен туннель:

1. Установите [ngrok](https://ngrok.com) и выполните `ngrok config add-authtoken <токен>`.
2. Запустите туннель: `ngrok http 5678`.
3. Скопируйте **https**-URL из вывода (например `https://xxxx.ngrok-free.app`).
4. В `.env` задайте `WEBHOOK_URL=https://ваш-url.ngrok-free.app` (и при желании `N8N_WEBHOOK_URL` тем же значением).
5. Перезапустите контейнеры: `docker compose down && docker compose up -d`.

После смены URL ngrok (например после перезапуска) обновите `WEBHOOK_URL` в `.env` и снова перезапустите n8n.

## 8) Импорт и запуск примера (Telegram Echo)

1. Импорт: в n8n `Workflows → Import from File` и выберите `workflows/telegram-echo-workflow.json`, либо попросите агента в Cursor импортировать workflow из папки через MCP (`n8n_create_workflow`).
2. В узлах **Telegram Trigger** и **Telegram Reply** создайте/выберите credential и вставьте токен бота (из [@BotFather](https://t.me/BotFather)).
3. Включите workflow (переключатель **Active**).
4. Напишите боту в Telegram — должен прийти ответ эхом.

## Остановка

```bash
docker compose down
```
