# Telegram Bot (Node.js)

Production-ready template README for a Node.js Telegram bot. It covers setup, environment, running locally with long polling, deploying with webhooks, and helpful tips.

## Features
- Minimal, clear setup with `.env`
- Works with long polling (local) or webhooks (production)
- Healthcheck endpoint and graceful shutdown (recommended)
- Structured scripts for dev/start/build

## Tech Stack
- Node.js 18+ (LTS recommended)
- Telegram Bot API
- Bot framework: Telegraf (recommended) or grammY

## Quick Start
1) Create a bot with BotFather in Telegram and copy your token.
2) Create a `.env` file:

```bash
BOT_TOKEN=123456:ABC-your-bot-token
# Optional, used for webhooks (see below)
PORT=3000
WEBHOOK_URL=https://your-domain.example.com
WEBHOOK_SECRET_PATH=/tgbot/secret-123
```

3) Install dependencies and run (examples assume Telegraf):

```bash
npm install telegraf
# Optional tooling
npm install -D nodemon typescript ts-node @types/node

# Dev (choose one depending on your setup)
npm run dev

# Start (production)
npm start
```

If you have not added scripts yet, see the Scripts section below for recommended entries.

## Minimal Example (Telegraf)
```js
// src/index.js
const { Telegraf } = require('telegraf');

const bot = new Telegraf(process.env.BOT_TOKEN);

bot.start((ctx) => ctx.reply('Welcome!'));
bot.help((ctx) => ctx.reply('Send me a sticker'));
bot.on('sticker', (ctx) => ctx.reply('👍'));
bot.hears('hi', (ctx) => ctx.reply('Hey there'));

// Long polling (local/dev)
if (!process.env.WEBHOOK_URL) {
  bot.launch();
  console.log('Bot started with long polling');
}

// Graceful stop
process.once('SIGINT', () => bot.stop('SIGINT'));
process.once('SIGTERM', () => bot.stop('SIGTERM'));
```

## Long Polling vs Webhooks
- Long polling: easiest locally; the bot pulls updates from Telegram.
- Webhooks: recommended for production; Telegram pushes updates to your HTTPS endpoint.

### Using Long Polling (Local)
- Ensure `WEBHOOK_URL` is NOT set in `.env`.
- Run `npm run dev` or `npm start`; you should see messages indicating the bot is polling.

### Using Webhooks (Production)
1) Set `WEBHOOK_URL` to your public HTTPS domain and choose a secret path, e.g. `/tgbot/secret-123`.
2) Expose an HTTP server that handles `POST WEBHOOK_SECRET_PATH` and passes updates to the bot.
3) Register the webhook once your service is up:

```bash
curl -X POST "https://api.telegram.org/bot${BOT_TOKEN}/setWebhook" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "'"${WEBHOOK_URL}${WEBHOOK_SECRET_PATH}"'",
    "drop_pending_updates": true
  }'
```

For Telegraf, a simple Express server example:
```js
// src/server.js
const express = require('express');
const { Telegraf } = require('telegraf');

const app = express();
app.use(express.json());

const bot = new Telegraf(process.env.BOT_TOKEN);
// register handlers as in index.js

const path = process.env.WEBHOOK_SECRET_PATH || '/tgbot';
app.post(path, (req, res) => {
  bot.handleUpdate(req.body, res);
});

app.get('/health', (_req, res) => res.status(200).send('ok'));

const port = process.env.PORT || 3000;
app.listen(port, async () => {
  console.log(`Listening on :${port}`);
});
```

Then set the webhook as shown above.

## Recommended package.json Scripts
Add or adapt as needed:
```json
{
  "scripts": {
    "dev": "nodemon src/index.js",
    "start": "node src/index.js",
    "start:webhook": "node src/server.js",
    "lint": "eslint .",
    "format": "prettier -w ."
  }
}
```

For TypeScript, use `ts-node` in dev and compile with `tsc` for production builds.

## Project Structure (suggested)
```text
.
├─ src/
│  ├─ index.js         # entry for long polling
│  ├─ server.js        # entry for webhook HTTP server
│  ├─ handlers/        # command/update handlers
│  └─ lib/             # utilities, api clients
├─ .env
├─ package.json
└─ README.md
```

## Environment Variables
- `BOT_TOKEN` (required): Telegram bot token from BotFather
- `PORT` (optional, default 3000): Port for webhook server
- `WEBHOOK_URL` (optional): Public HTTPS base URL for webhook
- `WEBHOOK_SECRET_PATH` (optional): Secret path suffix, e.g. `/tgbot/secret-123`

## Deployment Notes
- Ensure HTTPS and a stable public URL for webhooks.
- Popular options: Render, Railway, Fly.io, AWS Lambda + API Gateway, GCP Cloud Run, Azure App Service.
- Remember to set environment variables in your hosting platform.

## Testing
- Unit test handlers by passing mocked contexts.
- Consider integration tests using recorded Telegram update payloads.

## Security Best Practices
- Never commit `.env` or your `BOT_TOKEN`.
- Use a secret webhook path and validate source IPs if possible.
- Rate-limit public endpoints and add basic health checks.

## License
Choose a license (e.g., MIT) and include it here.
