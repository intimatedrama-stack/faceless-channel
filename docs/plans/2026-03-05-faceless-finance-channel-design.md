# Automated Faceless Finance Channel — Design Doc
**Date:** 2026-03-05
**Status:** Approved

## Overview
A fully automated, hands-off content engine that generates and publishes personal finance short-form videos to YouTube Shorts, Facebook Reels, and TikTok — once per day — with no human involvement after setup.

---

## Architecture

### Pipeline (Daily Cron — 9 AM)
```
[Cron Trigger]
    ↓
[GPT-4o] → Generate trending personal finance topic + hook + 3 key points
    ↓
[GPT-4o] → Write 55-60 sec Shorts script (hook → tips → CTA)
    ↓
[ElevenLabs API] → Convert script to MP3 voiceover (professional finance voice)
    ↓
[Pexels API] → Fetch 3-5 stock video clips matching finance keywords from script
    ↓
[Creatomate API] → Assemble: stock clips + voiceover + auto-captions + branding overlay
    ↓
[Parallel publish nodes]
  ├─ YouTube Data API v3 → YouTube Short (title, description, hashtags)
  ├─ Facebook Graph API → Facebook Reel to Facebook Page
  └─ TikTok API → TikTok upload
    ↓
[Google Sheets] → Log: topic, video URLs, timestamp, publish status
```

---

## Content Strategy

### Brand
- **Channel name:** WealthWise Daily (customizable)
- **Visual style:** Dark navy + gold overlays, Poppins Bold captions
- **Voice tone:** Confident, professional, slightly conversational

### Topic Categories (GPT rotates through)
| Category | Example |
|---|---|
| Budgeting | "The 50/30/20 rule that changed my finances" |
| Investing | "Why $100/month at 25 beats $1000/month at 35" |
| Credit | "3 moves that boosted my credit score 100 points" |
| Tax hacks | "Write-offs most people completely miss" |
| Mindset | "Habits of people who retire before 50" |
| Debt | "How to pay off $10K debt in 12 months" |

### Video Structure (60 seconds)
- **0-3s:** Hook — bold stat or question
- **3-50s:** 3 quick tips with on-screen text captions
- **50-60s:** CTA — "Follow for daily money tips"

---

## Tech Stack

| Tool | Purpose | Monthly Cost |
|---|---|---|
| n8n (self-hosted) | Workflow orchestration | Free |
| OpenAI GPT-4o | Topic + script generation | ~$5-10 |
| ElevenLabs Starter | AI voiceover MP3 | $5 |
| Pexels API | Free stock video footage | Free |
| Creatomate | Video assembly + rendering | $29 |
| YouTube Data API v3 | YouTube Shorts upload | Free |
| Facebook Graph API | Facebook Reels upload | Free |
| TikTok API | TikTok upload | Free |
| Google Sheets | Logging + topic tracking | Free |
| **Total** | | **~$39-44/mo** |

---

## Deliverables

1. **n8n workflow JSON** — importable, complete pipeline from trigger to publish
2. **Creatomate template JSON** — branded finance Shorts layout (navy/gold, captions)
3. **GPT system prompts** — topic generator + script writer prompts
4. **API setup guide** — step-by-step for all 6 API keys
5. **Google Sheets template** — topic queue + publish log
6. **n8n workflow: multi-platform publisher** — parallel YouTube + Facebook + TikTok nodes

---

## Monetization Path

| Timeline | Goal | Revenue |
|---|---|---|
| Month 1-3 | Build content library, grow | $0 (building) |
| Month 4-6 | YouTube YPP (1K subs + 10M Shorts views) | Ad revenue starts |
| Month 6+ | Finance CPM $10-25/1K views at 100K views/mo | $1,000-2,500/mo |
| Ongoing | Affiliate links (Robinhood, Acorns, Rocket Money) | $5-50/signup |

---

## Constraints & Notes
- Budget: $50-150/month (designed for ~$39-44/month)
- Platforms: YouTube Shorts + Facebook Reels + TikTok
- No dashboard — fully hands-off after initial setup
- Content: Personal Finance / Wealth niche
- Video length: 55-60 seconds (optimized for Shorts algorithm)
- n8n is self-hosted (already running for LeadLock project)
