# Faceless Channel (WealthWise Daily)

Automated personal-finance shorts pipeline: one video per day to YouTube Shorts, Facebook Reels, and TikTok after setup.

**Flow:** n8n cron → GPT topic/script → ElevenLabs voice → Pexels footage → Creatomate render → publish APIs → Google Sheets log.

## Docs

- [SETUP.md](SETUP.md) — API keys, n8n variables, first run
- [docs/plans/](docs/plans/) — design + implementation plan
- [prompts/](prompts/) — topic and script system prompts

## Quick start

1. Copy `.env.example` → `.env` and fill keys (OpenAI, ElevenLabs, Pexels, Creatomate, YouTube/FB/TikTok, Sheets).
2. Import the n8n workflow under `n8n/workflows/` (if present in your tree).
3. Follow [SETUP.md](SETUP.md) for OAuth and template IDs.

Monthly tooling cost is roughly $39–44 after free tiers.
