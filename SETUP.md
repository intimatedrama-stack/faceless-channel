# WealthWise Daily — Setup Guide

> Automated personal finance channel that posts 1 video/day to YouTube Shorts, Facebook Reels, and TikTok — zero manual work after setup.

**Estimated setup time:** 1-2 hours
**Monthly cost:** ~$39-44/month

---

## Step 1: Get All API Keys

### 1. OpenAI
1. Go to https://platform.openai.com/api-keys
2. Click "Create new secret key" → copy to `OPENAI_API_KEY`

### 2. ElevenLabs (Voice)
1. Go to https://elevenlabs.io → Sign up (free starter: 10K chars/mo)
2. Settings → API Key → copy to `ELEVENLABS_API_KEY`
3. Voice Library → search "Adam" or "Daniel" → click the voice
4. Copy the Voice ID from the URL or settings → paste to `ELEVENLABS_VOICE_ID`

### 3. Pexels (Stock Footage — Free)
1. Go to https://www.pexels.com/api/ → "Request API Key"
2. Fill in the form (instant approval) → copy key to `PEXELS_API_KEY`

### 4. Creatomate (Video Assembly — $29/mo)
1. Go to https://creatomate.com → Sign up
2. Settings → API Key → copy to `CREATOMATE_API_KEY`
3. **Create your video template:**
   - Go to Templates → New Template → "Start from JSON"
   - Paste the contents of `creatomate/finance-shorts-template.json` → "template" object
   - Save → copy the Template ID to `CREATOMATE_TEMPLATE_ID`

### 5. YouTube (OAuth2 — Free)
1. Go to https://console.cloud.google.com
2. Create a new project (e.g. "WealthWise Automation")
3. APIs & Services → Enable APIs → search "YouTube Data API v3" → Enable
4. Credentials → Create Credentials → OAuth 2.0 Client ID
5. Application type: Web application
6. Authorized redirect URI: Add your n8n instance URL + `/rest/oauth2-credential/callback`
7. Copy Client ID and Client Secret
8. **In n8n:** Settings → Credentials → New → YouTube OAuth2 API → enter ID + Secret → Connect (authorize)

### 6. Facebook (Free with Facebook Page)
1. Create a Facebook Page for your channel (if you don't have one)
2. Go to https://developers.facebook.com → Create App
3. Add "Pages" product
4. Use Graph API Explorer to generate a long-lived Page Access Token
5. Copy Page ID (from Page → About → Page ID) → `FACEBOOK_PAGE_ID`
6. Copy token → `FACEBOOK_ACCESS_TOKEN`

> ⚠️ Facebook tokens expire every 60 days. Set a calendar reminder to regenerate.

### 7. TikTok (Free)
1. Go to https://developers.tiktok.com
2. Register as developer → Create App
3. Request "Content Posting API" permission
4. Go through OAuth flow to get access token → `TIKTOK_ACCESS_TOKEN`

### 8. Google Sheets (Free)
1. Go to https://sheets.google.com → New spreadsheet
2. Rename to "WealthWise Daily Log"
3. Rename Tab 1 to "Log"
4. Add these headers in row 1: `Date | Topic Title | Category | YouTube URL | Facebook URL | TikTok URL | Status | Notes`
5. Copy Sheet ID from URL (`https://docs.google.com/spreadsheets/d/{SHEET_ID}/`) → `GOOGLE_SHEET_ID`
6. **In n8n:** Settings → Credentials → New → Google Sheets OAuth2 API → authorize

---

## Step 2: Set Up n8n Variables

In n8n: Settings → Variables → Create each variable:

| Name | Value |
|---|---|
| OPENAI_API_KEY | your OpenAI key |
| ELEVENLABS_API_KEY | your ElevenLabs key |
| ELEVENLABS_VOICE_ID | your voice ID |
| PEXELS_API_KEY | your Pexels key |
| CREATOMATE_API_KEY | your Creatomate key |
| CREATOMATE_TEMPLATE_ID | your template ID |
| FACEBOOK_PAGE_ID | your Facebook Page ID |
| FACEBOOK_ACCESS_TOKEN | your FB token |
| TIKTOK_ACCESS_TOKEN | your TikTok token |
| GOOGLE_SHEET_ID | your Sheet ID |

---

## Step 3: Import the Workflow

1. Open your n8n instance
2. Workflows → Import → Select `n8n/workflows/faceless-finance-daily.json`
3. The workflow opens with all nodes pre-configured

---

## Step 4: Test Run

1. Click "Execute Workflow" button (do NOT activate yet)
2. Watch each node turn green (~3 minutes total)
3. If any node turns red, hover it to see the error
4. Check your Google Sheets — a new row should appear
5. Check YouTube Studio, Facebook Creator Studio, TikTok Studio for the posted video

---

## Step 5: Activate

1. Once the test run succeeds, click the **Activate** toggle (top right)
2. The workflow will now run automatically every day at 9 AM
3. Monitor the first 3-5 runs in the Executions tab

---

## Monetization Next Steps

- **Month 3-4:** Add affiliate links to video descriptions (Robinhood: $5-20/signup, Acorns: $10/signup)
- **Month 4-6:** Apply for YouTube Partner Program (1K subs + 10M Shorts views)
- **Month 6+:** Finance CPM $10-25/1K views → $1,000-2,500/mo at 100K monthly views

---

## Common Issues

| Error | Fix |
|---|---|
| ElevenLabs 401 | API key or Voice ID wrong |
| Pexels no results | Query too specific — edit topic prompt |
| Creatomate render failed | Wrong template ID or variable name mismatch |
| YouTube 403 | Re-authorize OAuth2 credential in n8n |
| Facebook 190 | Token expired — regenerate long-lived token |
| TikTok 401 | Token expired — go through OAuth flow again |
