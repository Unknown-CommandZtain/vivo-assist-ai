# Deployment guide: vivo Assist AI

This walkthrough is for a *new* deployment. If you already have a running bot, keep its existing Worker, D1 database, and Telegram token; only update the source file, and do not run `/setup` unless reconnecting the webhook is necessary.

## A. Create the Telegram bot

1. In Telegram, open **[@BotFather](https://t.me/BotFather)**, send `/newbot`, and follow the prompts.
2. Save its bot token in a password manager. Do not add it to your GitHub repository.
3. For groups, configure the bot permissions you intend to use. Banning users and deleting messages require the appropriate Telegram administrator permissions.

## B. Create the Worker and D1 database

1. In the [Cloudflare dashboard](https://dash.cloudflare.com/), create a Worker, for example `vivo-assist-ai`.
2. Edit the Worker code to use **`src/worker.js`** from this repository and deploy.
3. Create a new Cloudflare **D1 database**, for example `vivo_assist_ai_db`.
4. Add a D1 binding to the Worker named **`DB`**, selecting that database. Save/deploy.
5. Record the exact Worker origin assigned by Cloudflare, e.g. `https://vivo-assist-ai.YOUR-SUBDOMAIN.workers.dev`.

**Do not reuse your old Realme bot's D1 database or webhook token.**

### Optional Wrangler deployment

For developers who use Wrangler instead of the dashboard, copy `wrangler.example.jsonc` to `wrangler.jsonc`, replace the Worker subdomain, admin ID, and D1 database UUID with values from your own Cloudflare account, then deploy with Wrangler. The sample is intentionally **not ready to deploy as-is**. `.gitignore` excludes the local config because it can contain account-specific information.

Never put real secrets in `wrangler.example.jsonc`. Configure secrets through the Cloudflare dashboard or `wrangler secret put`. You should not mix dashboard-only configuration with repo-based configuration without checking that deployments retain your bindings and triggers.

## C. Configure variables

Worker → Settings → **Variables and Secrets** (wording may vary):

| Name | Type | Example / notes |
| --- | --- | --- |
| `TELEGRAM_BOT_TOKEN` | Secret | Your actual new bot token; never print/share |
| `WEBHOOK_SECRET` | Secret | Generate with `python -c "import secrets; print(secrets.token_urlsafe(36))"` |
| `GEMINI_API_KEY` | Secret | Key from Google AI Studio |
| `SERPAPI_KEY` | Secret | Optional; required for `/lookup` and fresh search |
| `ADMIN_ID` | Text | Your numeric Telegram user ID, not your username |
| `WORKER_ORIGIN` | Text | Exact `https://...workers.dev` origin, no trailing slash |
| `GEMINI_MODEL` | Text | Optional; default currently `gemini-3.1-flash-lite` |
| `LIVE_SEARCH_ENABLED` | Text | Optional; `false` disables web search |
| `ALLOWED_GROUP_IDS` | Text | Optional; group chat IDs to initialize |

Keep `WEBHOOK_SECRET` private; it is also the secret value that verifies Telegram webhook requests. Use separate credentials for test deployments.

## D. Set up maintenance

Create a **Cron Trigger** with expression:

```text
* * * * *
```

This runs every minute for queue retries and maintenance. Without it, some failed or delayed requests might not be retried and cleanup won't run reliably.

## E. Initialize the database and webhook

1. Navigate to `https://YOUR-WORKER-ORIGIN/setup` using the *same origin configured in `WORKER_ORIGIN`*.
2. Enter `WEBHOOK_SECRET` (not the Telegram bot token).
3. Click **Initialize and connect**.
4. Then **Check status**. Look for `Webhook matches: true`.
5. Test `/start`, `/help`, `/latest`, `/islandmusic`, and `/privacy` in Telegram.

The `/setup` action creates D1 tables and registers the Telegram webhook. Only one service should own a Telegram bot token's webhook at a time. Do not run a separate polling deployment with the same token.

## F. Optional: BotFather command menu

Use `/setcommands` in BotFather and paste the entries in [`docs/COMMANDS.md`](COMMANDS.md) one per line in `command - Description` format, omitting `/` in command names. The Worker's owner-only `/reset` command registers a shorter chat-scoped menu; avoid it if you prefer the full public BotFather list.

## G. Upgrade an existing running bot

1. Back up your *currently deployed source* from the dashboard first.
2. Keep the existing `DB`, environment values, and Cron Trigger.
3. Replace the Worker source with `src/worker.js` and deploy.
4. Test `/help` and `/latest`. No BotFather update is needed for a formatting-only help-menu change.

**Do not publish production database dumps, webhook secrets, API keys, tokens, or private logs.**
