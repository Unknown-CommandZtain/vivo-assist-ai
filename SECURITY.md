# Security and responsible use

- Do **not** put real `TELEGRAM_BOT_TOKEN`, `WEBHOOK_SECRET`, `GEMINI_API_KEY`, `SERPAPI_KEY`, production D1 records, or private chat logs in public issues, screenshots, or commits.
- For live deployment, store API credentials using Cloudflare Worker **Secrets**; keep only non-sensitive settings as text values.
- Keep `WORKER_ORIGIN` equal to the exact deployment origin. Use a unique D1 database and Telegram token per independent bot.
- Telegram group moderation commands require the appropriate Telegram administrator permissions. Give the bot only the permissions it actually needs.
- The bot submits user text and, when enabled, photos to Google Gemini. Search requests and excerpts may also go to SerpApi and Gemini. Document this for your community and use `/privacy`.
- Search snippets and AI answers may be inaccurate or malicious. Check official documentation for releases, firmware and security guidance.
- If a secret was exposed, **revoke/rotate it at the provider immediately**. Removing it from only the most recent commit is not enough; investigate Git history and provider logs.
- If you discover a vulnerability, avoid posting exploit steps containing production credentials or personal data in a public issue. Contact the repository owner privately through an appropriate channel.

This repository contains no live secrets by design, but this is not an independent penetration test or a guarantee of security.
