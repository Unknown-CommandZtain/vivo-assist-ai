# vivo Assist AI 💙🤖

An **unofficial Telegram assistant for vivo and iQOO communities**, powered by Cloudflare Workers, Cloudflare D1, Google Gemini, and optional SerpApi web search.

> This is an independent community project. It is **not affiliated with, endorsed by, or operated by vivo or iQOO**. Device guides can differ by model, country, and firmware. Always verify software releases and firmware updates against official sources.

## What it does

- 🏝️ **OriginOS and Origin Island help:** `/island`, `/islandmusic`, `/originplayer`, `/originos`, `/funtouch` and `/region`.
- 📰 **Current-information questions:** `/latest` and other questions about new software releases can retrieve web search excerpts through SerpApi when configured. Search results are *not* guaranteed correct or complete.
- 🛠️ **Device and software troubleshooting:** notifications, background behavior, battery, charging, camera, gaming, updates, ROM differences and more.
- 🤖 **Gemini-powered Q&A and photo analysis** with configurable usage limits.
- 🧠 **Optional conversational memory:** up to six saved exchanges per user/chat/topic, used only within a 24-hour window.
- 🛡️ **Group tools:** owner-only warnings, bans/unbans, group enable/disable, polls, and optional auto-deletion of messages.
- 🔒 **Telegram webhook validation**, D1-backed job queue, retry and cleanup using a Cloudflare Cron Trigger.

## Quick start

The code runs as a **Cloudflare Worker** (not a Node.js Telegram polling process). Your Telegram bot token, API keys, owner ID, webhook secret, and Worker URL are configured separately; none are included in this repository.

1. Create a bot with [@BotFather](https://t.me/BotFather) and keep the token private.
2. Create a Cloudflare Worker and a **separate D1 database**, bound as `DB`.
3. Upload `src/worker.js` in the Cloudflare dashboard, or use the optional Wrangler configuration template in `wrangler.example.jsonc` (copy it to `wrangler.jsonc` and replace placeholders).
4. Configure the environment values below in your Worker's **Settings → Variables and Secrets**. Use **Secret** for credentials.
5. Add the Cron Trigger `* * * * *` (every minute).
6. Open `https://YOUR-WORKER-ORIGIN/setup`. Enter your `WEBHOOK_SECRET` to initialize the D1 tables and connect the Telegram webhook.
7. Try `/start`, `/help`, `/latest`, and `/islandmusic` in Telegram.

Full details: **[Deployment guide](docs/DEPLOYMENT.md)**.

### Environment values

| Variable / binding | Type | Purpose |
| --- | --- | --- |
| `DB` | D1 binding | Database for chats, jobs, preferences, history, warnings and cache |
| `TELEGRAM_BOT_TOKEN` | Secret | Token from BotFather |
| `WEBHOOK_SECRET` | Secret | Random 32–256-character string using `[A-Za-z0-9_-]` |
| `GEMINI_API_KEY` | Secret | Google Gemini API key |
| `SERPAPI_KEY` | Secret, optional | Needed for model lookup and live search |
| `ADMIN_ID` | Text | Your personal numeric Telegram user ID |
| `WORKER_ORIGIN` | Text | Exact deployed HTTPS Worker origin, **without** trailing slash |
| `GEMINI_MODEL` | Text, optional | Defaults in code to `gemini-3.1-flash-lite`; check model availability in your API account |
| `LIVE_SEARCH_ENABLED` | Text, optional | `false` disables live search (enabled by default) |
| `ALLOWED_GROUP_IDS` | Text, optional | Comma- or space-separated initial allowed group IDs |

`WEBHOOK_SECRET` is **not** the Telegram bot token. Never commit real `.dev.vars`, `.env`, Wrangler secrets, or database exports.

## Command reference

Open [Commands](docs/COMMANDS.md) or send `/help` to the bot. Highlights:

```text
/islandmusic   Unsupported music player / Origin Island guide
/latest        Verify the current major OriginOS version (uses search if available)
/lookup        Search vivo and iQOO device models
/updates       OTA rollout information
/games         Gaming and FPS tips
/privacy       Data-handling information
/forget        Clear this conversation's saved AI memory
/admin         Owner-only controls
```

## Architecture

```text
Telegram ── webhook + secret header ──> Cloudflare Worker
                                        ├── Cloudflare D1: jobs, settings, cache, memory
                                        ├── Google Gemini: AI / supported photo analysis
                                        ├── SerpApi: optional device + fresh information search
                                        └── Telegram Bot API: replies / moderation / polls
Cloudflare Cron (every minute) ────────> retries + maintenance
```

## Privacy and operation

User message content sent for AI answers is processed by Google Gemini. Queries requiring live search or device lookup may be sent to SerpApi. D1 stores pending jobs, usage records, user preferences, moderation history, and short-lived conversation history. The bot supports `/privacy`, `/forget`, and `/memory off`. Consult the providers' policies for retention outside this application. **Do not post personal data, credentials, or sensitive logs to public issues.**

The source includes a historical OriginOS 7 reference from 2026-11-10; that is *not* a claim it will always be the latest release. Live search depends on your key, quota, query routing and available public sources.

## Security

Please read [SECURITY.md](SECURITY.md). The sample configuration contains placeholders only. If you accidentally publish a real key, **rotate it immediately**, then remove it from the repository history as necessary.

## Project status

**v2.1, clean-help release.** Based on the author's Cloudflare Worker implementation. This repository provides the code and documentation; it does not provision accounts or deploy on your behalf.

**Licensing:** No license is included yet. Public visibility alone does not grant reuse rights. The repository owner can select and add an appropriate license when ready.
