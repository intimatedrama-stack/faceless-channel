# Automated Faceless Finance Channel Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a fully automated n8n pipeline that generates, assembles, and publishes one personal finance short-form video per day to YouTube Shorts, Facebook Reels, and TikTok — zero manual work after setup.

**Architecture:** n8n cron workflow runs daily at 9 AM: GPT-4o generates topic + 60-second script → ElevenLabs converts to MP3 voiceover → Pexels provides free stock finance footage → Creatomate renders the final branded video → YouTube/Facebook/TikTok APIs auto-publish → Google Sheets logs every run.

**Tech Stack:** n8n (self-hosted), OpenAI GPT-4o API, ElevenLabs API v1, Pexels API v1, Creatomate API, YouTube Data API v3, Facebook Graph API v21, TikTok Content Posting API, Google Sheets API v4

---

## Project Structure

```
faceless-channel/
├── docs/plans/                    # Design + implementation docs
├── n8n/
│   └── workflows/
│       └── faceless-finance-daily.json   # Main importable n8n workflow
├── prompts/
│   ├── topic-generator.md                # GPT system prompt for topics
│   └── script-writer.md                  # GPT system prompt for scripts
├── creatomate/
│   └── finance-shorts-template.json      # Creatomate video template
├── sheets/
│   └── setup.md                          # Google Sheets column guide
├── .env.example                          # All API keys template
└── SETUP.md                              # Step-by-step setup guide
```

---

## Task 1: Project Scaffolding

**Files:**
- Create: `C:\Users\intim\Desktop\faceless-channel\.env.example`
- Create: `C:\Users\intim\Desktop\faceless-channel\SETUP.md`

**Step 1: Create .env.example with all required API keys**

Create `C:\Users\intim\Desktop\faceless-channel\.env.example`:

```bash
# OpenAI
OPENAI_API_KEY=sk-...

# ElevenLabs
ELEVENLABS_API_KEY=...
ELEVENLABS_VOICE_ID=...   # Get from: https://elevenlabs.io/voice-library

# Pexels
PEXELS_API_KEY=...         # Free at: https://www.pexels.com/api/

# Creatomate
CREATOMATE_API_KEY=...
CREATOMATE_TEMPLATE_ID=... # Set after Task 3

# YouTube (OAuth2 via n8n credential)
# Set up in n8n UI: Credentials → New → YouTube OAuth2 API

# Facebook
FACEBOOK_PAGE_ID=...
FACEBOOK_ACCESS_TOKEN=...  # Long-lived page token

# TikTok
TIKTOK_ACCESS_TOKEN=...    # From TikTok Developer Portal

# Google Sheets
GOOGLE_SHEET_ID=...        # From sheet URL: /spreadsheets/d/{ID}/
GOOGLE_SHEETS_TAB=Log      # Sheet tab name
```

**Step 2: Create SETUP.md**

Create `C:\Users\intim\Desktop\faceless-channel\SETUP.md`:

```markdown
# WealthWise Daily — Setup Guide

## Step 1: API Keys (get these first)

### OpenAI
1. Go to https://platform.openai.com/api-keys
2. Create new secret key → copy to OPENAI_API_KEY

### ElevenLabs
1. Go to https://elevenlabs.io → Sign up (free tier: 10K chars/month)
2. Settings → API Key → copy to ELEVENLABS_API_KEY
3. Voice Library → pick "Adam" or "Daniel" (professional finance voice)
4. Click the voice → copy Voice ID to ELEVENLABS_VOICE_ID

### Pexels
1. Go to https://www.pexels.com/api/ → Request API Key (free)
2. Copy to PEXELS_API_KEY

### Creatomate
1. Go to https://creatomate.com → Sign up ($29/mo for 100 renders)
2. Settings → API Key → copy to CREATOMATE_API_KEY
3. After Task 3: create template → copy Template ID to CREATOMATE_TEMPLATE_ID

### YouTube
1. Go to https://console.cloud.google.com
2. Create project → Enable "YouTube Data API v3"
3. Create OAuth2 credentials (Web application)
4. In n8n: Credentials → New → YouTube OAuth2 API → enter Client ID + Secret
5. Authorize by clicking "Connect" (grants upload access)

### Facebook
1. Go to https://developers.facebook.com → Create App
2. Add "Pages API" product
3. Generate a long-lived Page Access Token for your Finance page
4. Copy Page ID and Access Token

### TikTok
1. Go to https://developers.tiktok.com → Register Developer
2. Create App → request "Content Posting API" access
3. Generate access token → copy to TIKTOK_ACCESS_TOKEN

### Google Sheets
1. Create a new Google Sheet
2. Name first tab "Log"
3. Add headers in row 1: Date | Topic | Script | YouTube URL | Facebook URL | TikTok URL | Status
4. Copy sheet ID from URL to GOOGLE_SHEET_ID

## Step 2: n8n Setup
1. Open your n8n instance
2. Go to Workflows → Import From File
3. Select: n8n/workflows/faceless-finance-daily.json
4. Open the workflow → add credentials for each HTTP node (see credential setup above)
5. Update environment variables in n8n Settings → Variables

## Step 3: Activate
1. Open the workflow in n8n
2. Click "Activate" toggle (top right)
3. It will run daily at 9 AM automatically
4. Check Google Sheets log after first run
```

**Step 3: Verify structure**

Run: `ls /c/Users/intim/Desktop/faceless-channel/`
Expected: See `docs/`, `n8n/`, `prompts/`, `creatomate/`, `sheets/`, `.env.example`, `SETUP.md`

---

## Task 2: GPT System Prompts

**Files:**
- Create: `prompts/topic-generator.md`
- Create: `prompts/script-writer.md`

**Step 1: Create topic generator prompt**

Create `C:\Users\intim\Desktop\faceless-channel\prompts\topic-generator.md`:

```markdown
# Topic Generator — System Prompt

Use this as the `system` message in the OpenAI API call.

---

You are a viral personal finance content strategist for YouTube Shorts.

Generate ONE trending personal finance topic for a 60-second short video.
The topic must be: specific, actionable, and relatable to ages 22-45.

Rotate through these categories (pick the one least recently used):
- Budgeting (50/30/20, zero-based, no-spend challenges)
- Investing (compound interest, index funds, ETFs, 401k)
- Credit scores (hacks, mistakes, repair)
- Tax optimization (write-offs, deductions, credits)
- Debt payoff (avalanche vs snowball, consolidation)
- Wealth mindset (habits of millionaires, money psychology)
- Side income (passive income, cash flow, monetization)

Return ONLY valid JSON, no markdown, no explanation:
{
  "title": "Catchy video title under 60 chars",
  "hook": "Opening sentence that creates curiosity or urgency (under 15 words)",
  "keywords": ["keyword1", "keyword2", "keyword3"],
  "category": "budgeting|investing|credit|tax|debt|mindset|income",
  "pexels_query": "stock footage search term (e.g. 'money investment finance')"
}
```

**Step 2: Create script writer prompt**

Create `C:\Users\intim\Desktop\faceless-channel\prompts\script-writer.md`:

```markdown
# Script Writer — System Prompt

Use this as the `system` message in the second OpenAI API call.

---

You are a viral short-form video scriptwriter specializing in personal finance content.

Write a 55-60 second spoken script for YouTube Shorts / TikTok based on the topic JSON provided.

Rules:
- Hook: First 3 seconds must stop the scroll. Start with a shocking stat, bold claim, or direct question.
- Use simple language (8th grade reading level)
- 3 concrete, actionable tips (numbered)
- End with CTA: "Follow for a daily money tip"
- NO filler words: "um", "uh", "you know"
- Speak directly to viewer using "you" and "your"
- Total word count: 130-160 words (reads at ~60 seconds)

Return ONLY the spoken script text, no stage directions, no JSON, no title.
Start directly with the hook sentence.
```

---

## Task 3: Creatomate Video Template

**Files:**
- Create: `creatomate/finance-shorts-template.json`

**What Creatomate does:** Takes your template + dynamic variables (video clips, audio file, caption text) and renders a finished MP4. You set the template up ONCE on Creatomate's website, then call their API with variables.

**Step 1: Create the template definition file**

Create `C:\Users\intim\Desktop\faceless-channel\creatomate\finance-shorts-template.json`:

```json
{
  "comment": "Upload this template structure to Creatomate at https://creatomate.com/templates/new",
  "comment2": "After creating, copy the Template ID to .env CREATOMATE_TEMPLATE_ID",
  "template": {
    "output_format": "mp4",
    "width": 1080,
    "height": 1920,
    "duration": 60,
    "elements": [
      {
        "id": "background_video",
        "type": "video",
        "source": "{{VIDEO_URL}}",
        "fit": "cover",
        "volume": 0,
        "width": "100%",
        "height": "100%",
        "time": 0,
        "duration": 60
      },
      {
        "id": "dark_overlay",
        "type": "rectangle",
        "width": "100%",
        "height": "100%",
        "fill_color": "rgba(0,0,0,0.45)",
        "time": 0,
        "duration": 60
      },
      {
        "id": "voiceover",
        "type": "audio",
        "source": "{{AUDIO_URL}}",
        "time": 0
      },
      {
        "id": "background_music",
        "type": "audio",
        "source": "https://cdn.creatomate.com/demo/music/corporate-background.mp3",
        "volume": 0.08,
        "time": 0,
        "duration": 60
      },
      {
        "id": "channel_brand",
        "type": "text",
        "text": "WealthWise Daily",
        "font_family": "Poppins",
        "font_weight": "700",
        "font_size": 32,
        "fill_color": "#FFD700",
        "x_alignment": "center",
        "y": "92%",
        "time": 0,
        "duration": 60,
        "animations": [
          {"type": "fade", "time": 0, "duration": 0.5}
        ]
      },
      {
        "id": "caption_block",
        "type": "text",
        "text": "{{CAPTION_TEXT}}",
        "font_family": "Poppins",
        "font_weight": "800",
        "font_size": 52,
        "fill_color": "#FFFFFF",
        "stroke_color": "#000000",
        "stroke_width": 3,
        "width": "88%",
        "x_alignment": "center",
        "y": "55%",
        "word_wrap": true,
        "animations": [
          {"type": "typewriter", "time": 0, "duration": 60}
        ]
      },
      {
        "id": "hook_banner",
        "type": "rectangle",
        "width": "100%",
        "height": "12%",
        "fill_color": "rgba(0, 31, 63, 0.85)",
        "y": "10%",
        "time": 0,
        "duration": 8
      },
      {
        "id": "hook_text",
        "type": "text",
        "text": "{{HOOK_TEXT}}",
        "font_family": "Poppins",
        "font_weight": "700",
        "font_size": 38,
        "fill_color": "#FFD700",
        "width": "90%",
        "x_alignment": "center",
        "y": "10%",
        "word_wrap": true,
        "time": 0,
        "duration": 8
      }
    ]
  },
  "variables": {
    "VIDEO_URL": "URL of Pexels stock video clip",
    "AUDIO_URL": "URL of ElevenLabs MP3 voiceover",
    "CAPTION_TEXT": "First 100 chars of script (hook sentence)",
    "HOOK_TEXT": "Hook sentence from topic JSON"
  },
  "setup_instructions": [
    "1. Go to https://creatomate.com/templates",
    "2. Click 'New Template' → 'From JSON'",
    "3. Paste the 'template' object above",
    "4. Save → copy Template ID to .env CREATOMATE_TEMPLATE_ID",
    "5. Optional: adjust colors/fonts in the visual editor"
  ]
}
```

**Step 2: Verify template file**

Run: `cat /c/Users/intim/Desktop/faceless-channel/creatomate/finance-shorts-template.json`
Expected: Valid JSON, no parse errors

---

## Task 4: Google Sheets Tracking Setup

**Files:**
- Create: `sheets/setup.md`

**Step 1: Create setup instructions**

Create `C:\Users\intim\Desktop\faceless-channel\sheets\setup.md`:

```markdown
# Google Sheets Tracking Template

## Create the Sheet

1. Go to https://sheets.google.com → New spreadsheet
2. Rename it: "WealthWise Daily Log"
3. Rename Tab 1 to: "Log"
4. Add these headers in row 1 (one per column):

| A | B | C | D | E | F | G | H |
|---|---|---|---|---|---|---|---|
| Date | Topic Title | Category | YouTube URL | Facebook URL | TikTok URL | Status | Notes |

5. Format row 1: Bold, dark navy background (#001F3F), white text (#FFFFFF)
6. Freeze row 1: View → Freeze → 1 row

## Get Sheet ID

From the URL: https://docs.google.com/spreadsheets/d/{SHEET_ID}/edit
Copy the {SHEET_ID} portion to your .env file.

## Google Sheets API Setup (for n8n)

1. Go to https://console.cloud.google.com
2. Enable "Google Sheets API" on same project as YouTube
3. In n8n: Credentials → New → Google Sheets OAuth2 API
4. Authorize with your Google account
```

---

## Task 5: n8n Workflow — Core Pipeline

**Files:**
- Create: `n8n/workflows/faceless-finance-daily.json`

This is the main deliverable. A complete importable n8n workflow JSON.

**Step 1: Create the complete n8n workflow JSON**

Create `C:\Users\intim\Desktop\faceless-channel\n8n\workflows\faceless-finance-daily.json`:

```json
{
  "name": "WealthWise Daily — Faceless Finance Automation",
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "cronExpression",
              "expression": "0 9 * * *"
            }
          ]
        }
      },
      "id": "node-trigger",
      "name": "Daily 9AM Trigger",
      "type": "n8n-nodes-base.scheduleTrigger",
      "position": [240, 400]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.openai.com/v1/chat/completions",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Authorization",
              "value": "Bearer {{$vars.OPENAI_API_KEY}}"
            },
            {
              "name": "Content-Type",
              "value": "application/json"
            }
          ]
        },
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "model",
              "value": "gpt-4o"
            },
            {
              "name": "temperature",
              "value": "0.85"
            },
            {
              "name": "response_format",
              "value": "{\"type\": \"json_object\"}"
            },
            {
              "name": "messages",
              "value": "[{\"role\":\"system\",\"content\":\"You are a viral personal finance content strategist for YouTube Shorts.\\n\\nGenerate ONE trending personal finance topic for a 60-second short video.\\nThe topic must be: specific, actionable, and relatable to ages 22-45.\\n\\nRotate through these categories (pick randomly):\\n- Budgeting (50/30/20, zero-based, no-spend challenges)\\n- Investing (compound interest, index funds, ETFs, 401k)\\n- Credit scores (hacks, mistakes, repair)\\n- Tax optimization (write-offs, deductions, credits)\\n- Debt payoff (avalanche vs snowball, consolidation)\\n- Wealth mindset (habits of millionaires, money psychology)\\n- Side income (passive income, cash flow, monetization)\\n\\nReturn ONLY valid JSON:\\n{\\n  \\\"title\\\": \\\"Catchy video title under 60 chars\\\",\\n  \\\"hook\\\": \\\"Opening sentence that creates curiosity or urgency (under 15 words)\\\",\\n  \\\"keywords\\\": [\\\"keyword1\\\", \\\"keyword2\\\", \\\"keyword3\\\"],\\n  \\\"category\\\": \\\"budgeting|investing|credit|tax|debt|mindset|income\\\",\\n  \\\"pexels_query\\\": \\\"stock footage search term (e.g. 'money finance wealth')\\\"\\n}\"},{\"role\":\"user\",\"content\":\"Generate today's finance video topic.\"}]"
            }
          ]
        },
        "options": {}
      },
      "id": "node-generate-topic",
      "name": "Generate Topic (GPT-4o)",
      "type": "n8n-nodes-base.httpRequest",
      "position": [480, 400]
    },
    {
      "parameters": {
        "jsCode": "const raw = $input.first().json.choices[0].message.content;\nconst topic = JSON.parse(raw);\nreturn [{ json: topic }];"
      },
      "id": "node-parse-topic",
      "name": "Parse Topic JSON",
      "type": "n8n-nodes-base.code",
      "position": [720, 400]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.openai.com/v1/chat/completions",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Authorization",
              "value": "Bearer {{$vars.OPENAI_API_KEY}}"
            },
            {
              "name": "Content-Type",
              "value": "application/json"
            }
          ]
        },
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "model",
              "value": "gpt-4o"
            },
            {
              "name": "temperature",
              "value": "0.7"
            },
            {
              "name": "messages",
              "value": "={{ [{\"role\":\"system\",\"content\":\"You are a viral short-form video scriptwriter specializing in personal finance.\\n\\nWrite a 55-60 second spoken script for YouTube Shorts / TikTok.\\n\\nRules:\\n- Hook: First 3 seconds must stop the scroll. Start with a shocking stat or bold claim.\\n- Use simple language (8th grade reading level)\\n- 3 concrete, actionable tips (numbered)\\n- End with CTA: Follow for a daily money tip\\n- NO filler words\\n- Speak directly to viewer using you and your\\n- Total word count: 130-160 words\\n\\nReturn ONLY the spoken script text. Start directly with the hook sentence.\"},{\"role\":\"user\",\"content\":\"Write a script for this topic: \" + $json.title + \"\\nHook: \" + $json.hook + \"\\nCategory: \" + $json.category}] }}"
            }
          ]
        },
        "options": {}
      },
      "id": "node-generate-script",
      "name": "Generate Script (GPT-4o)",
      "type": "n8n-nodes-base.httpRequest",
      "position": [960, 400]
    },
    {
      "parameters": {
        "jsCode": "const topic = $('Parse Topic JSON').first().json;\nconst script = $input.first().json.choices[0].message.content.trim();\nreturn [{ json: { ...topic, script } }];"
      },
      "id": "node-merge-script",
      "name": "Merge Script + Topic",
      "type": "n8n-nodes-base.code",
      "position": [1200, 400]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.elevenlabs.io/v1/text-to-speech/{{$vars.ELEVENLABS_VOICE_ID}}",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "xi-api-key",
              "value": "{{$vars.ELEVENLABS_API_KEY}}"
            },
            {
              "name": "Content-Type",
              "value": "application/json"
            },
            {
              "name": "Accept",
              "value": "audio/mpeg"
            }
          ]
        },
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "text",
              "value": "={{ $json.script }}"
            },
            {
              "name": "model_id",
              "value": "eleven_turbo_v2"
            },
            {
              "name": "voice_settings",
              "value": "{\"stability\":0.5,\"similarity_boost\":0.75,\"style\":0.3,\"use_speaker_boost\":true}"
            }
          ]
        },
        "options": {
          "response": {
            "response": {
              "responseFormat": "file",
              "outputPropertyName": "audio"
            }
          }
        }
      },
      "id": "node-elevenlabs",
      "name": "Generate Voiceover (ElevenLabs)",
      "type": "n8n-nodes-base.httpRequest",
      "position": [1440, 400]
    },
    {
      "parameters": {
        "jsCode": "// Upload audio binary to a temp file host\n// We use transfer.sh (free, no signup) for temp file hosting\n// Audio is in binary format from previous node\nconst allData = $input.all();\nconst audioNode = allData[0];\n\n// Pass through all context plus binary audio data\nconst topicData = $('Merge Script + Topic').first().json;\nreturn [{ json: topicData, binary: audioNode.binary }];"
      },
      "id": "node-prep-upload",
      "name": "Prepare Audio Upload",
      "type": "n8n-nodes-base.code",
      "position": [1680, 400]
    },
    {
      "parameters": {
        "method": "PUT",
        "url": "https://transfer.sh/voiceover-{{$now.format('YYYY-MM-DD')}}.mp3",
        "sendBody": true,
        "bodyParameters": {
          "parameters": []
        },
        "options": {
          "response": {
            "response": {
              "responseFormat": "text"
            }
          }
        },
        "sendBinaryData": true,
        "binaryPropertyName": "audio"
      },
      "id": "node-upload-audio",
      "name": "Upload Audio to transfer.sh",
      "type": "n8n-nodes-base.httpRequest",
      "position": [1920, 400],
      "notes": "Uploads MP3 to temporary public URL for Creatomate. File expires after 14 days."
    },
    {
      "parameters": {
        "jsCode": "const audioUrl = $input.first().json.data.trim();\nconst topicData = $('Merge Script + Topic').first().json;\nreturn [{ json: { ...topicData, audioUrl } }];"
      },
      "id": "node-save-audio-url",
      "name": "Save Audio URL",
      "type": "n8n-nodes-base.code",
      "position": [2160, 400]
    },
    {
      "parameters": {
        "method": "GET",
        "url": "=https://api.pexels.com/videos/search?query={{encodeURIComponent($json.pexels_query)}}&per_page=5&orientation=portrait&size=medium",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Authorization",
              "value": "{{$vars.PEXELS_API_KEY}}"
            }
          ]
        },
        "options": {}
      },
      "id": "node-pexels",
      "name": "Fetch Stock Footage (Pexels)",
      "type": "n8n-nodes-base.httpRequest",
      "position": [2400, 400]
    },
    {
      "parameters": {
        "jsCode": "// Pick the first HD video file from the first result\nconst videos = $input.first().json.videos;\nif (!videos || videos.length === 0) {\n  throw new Error('No Pexels videos found for query');\n}\n\nconst video = videos[0];\n// Prefer HD (1280x720) or closest\nconst files = video.video_files;\nconst hd = files.find(f => f.quality === 'hd') || files[0];\n\nconst topicData = $('Save Audio URL').first().json;\nreturn [{ json: { ...topicData, videoUrl: hd.link } }];"
      },
      "id": "node-pick-video",
      "name": "Pick Best Video Clip",
      "type": "n8n-nodes-base.code",
      "position": [2640, 400]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://api.creatomate.com/v1/renders",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Authorization",
              "value": "Bearer {{$vars.CREATOMATE_API_KEY}}"
            },
            {
              "name": "Content-Type",
              "value": "application/json"
            }
          ]
        },
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "template_id",
              "value": "{{$vars.CREATOMATE_TEMPLATE_ID}}"
            },
            {
              "name": "modifications",
              "value": "={{ { \"VIDEO_URL\": $json.videoUrl, \"AUDIO_URL\": $json.audioUrl, \"HOOK_TEXT\": $json.hook, \"CAPTION_TEXT\": $json.script.substring(0, 120) } }}"
            }
          ]
        },
        "options": {}
      },
      "id": "node-creatomate-render",
      "name": "Render Video (Creatomate)",
      "type": "n8n-nodes-base.httpRequest",
      "position": [2880, 400]
    },
    {
      "parameters": {
        "jsCode": "const renders = $input.first().json;\nconst renderId = Array.isArray(renders) ? renders[0].id : renders.id;\nconst topicData = $('Pick Best Video Clip').first().json;\nreturn [{ json: { ...topicData, renderId } }];"
      },
      "id": "node-save-render-id",
      "name": "Save Render ID",
      "type": "n8n-nodes-base.code",
      "position": [3120, 400]
    },
    {
      "parameters": {
        "amount": 90,
        "unit": "seconds"
      },
      "id": "node-wait-render",
      "name": "Wait 90s for Render",
      "type": "n8n-nodes-base.wait",
      "position": [3360, 400],
      "notes": "Creatomate typically renders a 60-second video in 30-90 seconds."
    },
    {
      "parameters": {
        "method": "GET",
        "url": "=https://api.creatomate.com/v1/renders/{{$json.renderId}}",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Authorization",
              "value": "Bearer {{$vars.CREATOMATE_API_KEY}}"
            }
          ]
        },
        "options": {}
      },
      "id": "node-check-render",
      "name": "Check Render Status",
      "type": "n8n-nodes-base.httpRequest",
      "position": [3600, 400]
    },
    {
      "parameters": {
        "jsCode": "const render = $input.first().json;\nconst status = render.status;\n\nif (status === 'succeeded') {\n  const topicData = $('Save Render ID').first().json;\n  return [{ json: { ...topicData, videoFileUrl: render.url, renderStatus: 'succeeded' } }];\n} else if (status === 'failed') {\n  throw new Error('Render failed: ' + JSON.stringify(render));\n} else {\n  // Still rendering\n  throw new Error('STILL_RENDERING:' + status);\n}"
      },
      "id": "node-check-status",
      "name": "Extract Video URL",
      "type": "n8n-nodes-base.code",
      "position": [3840, 400],
      "notes": "If render is not done, error is caught — add retry logic if needed"
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://www.googleapis.com/upload/youtube/v3/videos?uploadType=multipart&part=snippet,status",
        "authentication": "oAuth2",
        "credentialType": "youTubeOAuth2Api",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Content-Type",
              "value": "application/json"
            }
          ]
        },
        "sendBody": true,
        "contentType": "multipart-form-data",
        "bodyParameters": {
          "parameters": [
            {
              "parameterType": "formBinaryData",
              "name": "video",
              "inputDataFieldName": "videoFile"
            },
            {
              "name": "metadata",
              "value": "={{ JSON.stringify({ snippet: { title: $json.title + ' #Shorts', description: $json.script + '\\n\\n#personalfinance #money #finance #wealth #investing #budgeting #financetips #shorts', tags: [...$json.keywords, 'personal finance', 'money tips', 'shorts', 'finance'], categoryId: '27', defaultLanguage: 'en' }, status: { privacyStatus: 'public', selfDeclaredMadeForKids: false } }) }}"
            }
          ]
        },
        "options": {}
      },
      "id": "node-youtube-upload",
      "name": "Upload to YouTube Shorts",
      "type": "n8n-nodes-base.httpRequest",
      "position": [4080, 260],
      "notes": "Requires YouTube OAuth2 credential in n8n. Video auto-qualifies as Short if <= 60s and 9:16 aspect ratio."
    },
    {
      "parameters": {
        "method": "POST",
        "url": "=https://graph.facebook.com/v21.0/{{$vars.FACEBOOK_PAGE_ID}}/video_reels",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Content-Type",
              "value": "application/json"
            }
          ]
        },
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "upload_phase",
              "value": "start"
            },
            {
              "name": "access_token",
              "value": "{{$vars.FACEBOOK_ACCESS_TOKEN}}"
            }
          ]
        },
        "options": {}
      },
      "id": "node-facebook-start",
      "name": "Facebook Reel — Start Upload",
      "type": "n8n-nodes-base.httpRequest",
      "position": [4080, 400]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "=https://rupload.facebook.com/video-upload/v21.0/{{$input.first().json.video_id}}/",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Authorization",
              "value": "OAuth {{$vars.FACEBOOK_ACCESS_TOKEN}}"
            },
            {
              "name": "offset",
              "value": "0"
            },
            {
              "name": "file_size",
              "value": "={{ $('Extract Video URL').first().json.videoFileUrl }}"
            }
          ]
        },
        "sendBody": true,
        "sendBinaryData": true,
        "binaryPropertyName": "videoFile",
        "options": {}
      },
      "id": "node-facebook-upload",
      "name": "Facebook Reel — Upload Video",
      "type": "n8n-nodes-base.httpRequest",
      "position": [4080, 540]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "=https://graph.facebook.com/v21.0/{{$vars.FACEBOOK_PAGE_ID}}/video_reels",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Content-Type",
              "value": "application/json"
            }
          ]
        },
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "upload_phase",
              "value": "finish"
            },
            {
              "name": "video_id",
              "value": "={{ $('Facebook Reel — Start Upload').first().json.video_id }}"
            },
            {
              "name": "access_token",
              "value": "{{$vars.FACEBOOK_ACCESS_TOKEN}}"
            },
            {
              "name": "title",
              "value": "={{ $('Extract Video URL').first().json.title }}"
            },
            {
              "name": "description",
              "value": "={{ $('Extract Video URL').first().json.script + '\\n\\n#personalfinance #money #finance #wealth' }}"
            },
            {
              "name": "published",
              "value": "true"
            }
          ]
        },
        "options": {}
      },
      "id": "node-facebook-publish",
      "name": "Facebook Reel — Publish",
      "type": "n8n-nodes-base.httpRequest",
      "position": [4080, 680]
    },
    {
      "parameters": {
        "method": "POST",
        "url": "https://open.tiktokapis.com/v2/post/publish/video/init/",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "Authorization",
              "value": "Bearer {{$vars.TIKTOK_ACCESS_TOKEN}}"
            },
            {
              "name": "Content-Type",
              "value": "application/json; charset=UTF-8"
            }
          ]
        },
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "post_info",
              "value": "={{ { title: $('Extract Video URL').first().json.title + ' #personalfinance #money', privacy_level: 'PUBLIC_TO_EVERYONE', disable_duet: false, disable_comment: false, disable_stitch: false, video_cover_timestamp_ms: 1000 } }}"
            },
            {
              "name": "source_info",
              "value": "={{ { source: 'PULL_FROM_URL', video_url: $('Extract Video URL').first().json.videoFileUrl } }}"
            }
          ]
        },
        "options": {}
      },
      "id": "node-tiktok-upload",
      "name": "Upload to TikTok",
      "type": "n8n-nodes-base.httpRequest",
      "position": [4080, 820]
    },
    {
      "parameters": {
        "jsCode": "const topic = $('Extract Video URL').first().json;\nconst ytResult = $('Upload to YouTube Shorts').first().json;\nconst fbResult = $('Facebook Reel — Publish').first().json;\nconst ttResult = $('Upload to TikTok').first().json;\n\nconst ytVideoId = ytResult.id || 'upload_failed';\nconst fbPostId = fbResult.post_id || fbResult.video_id || 'upload_failed';\nconst ttPublishId = ttResult.data?.publish_id || 'upload_failed';\n\nconst ytUrl = ytVideoId !== 'upload_failed' ? `https://youtube.com/shorts/${ytVideoId}` : 'FAILED';\nconst fbUrl = fbPostId !== 'upload_failed' ? `https://facebook.com/${fbPostId}` : 'FAILED';\n\nreturn [{\n  json: {\n    date: new Date().toISOString().split('T')[0],\n    title: topic.title,\n    category: topic.category,\n    youtubeUrl: ytUrl,\n    facebookUrl: fbUrl,\n    tiktokPublishId: ttPublishId,\n    status: (ytVideoId !== 'upload_failed') ? 'SUCCESS' : 'PARTIAL',\n    script: topic.script\n  }\n}];"
      },
      "id": "node-compile-results",
      "name": "Compile Publish Results",
      "type": "n8n-nodes-base.code",
      "position": [4320, 400]
    },
    {
      "parameters": {
        "operation": "append",
        "documentId": {
          "value": "={{$vars.GOOGLE_SHEET_ID}}"
        },
        "sheetName": "Log",
        "columns": {
          "mappingMode": "defineBelow",
          "value": {
            "Date": "={{$json.date}}",
            "Topic Title": "={{$json.title}}",
            "Category": "={{$json.category}}",
            "YouTube URL": "={{$json.youtubeUrl}}",
            "Facebook URL": "={{$json.facebookUrl}}",
            "TikTok URL": "={{$json.tiktokPublishId}}",
            "Status": "={{$json.status}}",
            "Notes": ""
          }
        },
        "options": {}
      },
      "id": "node-log-sheets",
      "name": "Log to Google Sheets",
      "type": "n8n-nodes-base.googleSheets",
      "position": [4560, 400]
    }
  ],
  "connections": {
    "Daily 9AM Trigger": {
      "main": [[{"node": "Generate Topic (GPT-4o)", "type": "main", "index": 0}]]
    },
    "Generate Topic (GPT-4o)": {
      "main": [[{"node": "Parse Topic JSON", "type": "main", "index": 0}]]
    },
    "Parse Topic JSON": {
      "main": [[{"node": "Generate Script (GPT-4o)", "type": "main", "index": 0}]]
    },
    "Generate Script (GPT-4o)": {
      "main": [[{"node": "Merge Script + Topic", "type": "main", "index": 0}]]
    },
    "Merge Script + Topic": {
      "main": [[{"node": "Generate Voiceover (ElevenLabs)", "type": "main", "index": 0}]]
    },
    "Generate Voiceover (ElevenLabs)": {
      "main": [[{"node": "Prepare Audio Upload", "type": "main", "index": 0}]]
    },
    "Prepare Audio Upload": {
      "main": [[{"node": "Upload Audio to transfer.sh", "type": "main", "index": 0}]]
    },
    "Upload Audio to transfer.sh": {
      "main": [[{"node": "Save Audio URL", "type": "main", "index": 0}]]
    },
    "Save Audio URL": {
      "main": [[{"node": "Fetch Stock Footage (Pexels)", "type": "main", "index": 0}]]
    },
    "Fetch Stock Footage (Pexels)": {
      "main": [[{"node": "Pick Best Video Clip", "type": "main", "index": 0}]]
    },
    "Pick Best Video Clip": {
      "main": [[{"node": "Render Video (Creatomate)", "type": "main", "index": 0}]]
    },
    "Render Video (Creatomate)": {
      "main": [[{"node": "Save Render ID", "type": "main", "index": 0}]]
    },
    "Save Render ID": {
      "main": [[{"node": "Wait 90s for Render", "type": "main", "index": 0}]]
    },
    "Wait 90s for Render": {
      "main": [[{"node": "Check Render Status", "type": "main", "index": 0}]]
    },
    "Check Render Status": {
      "main": [[{"node": "Extract Video URL", "type": "main", "index": 0}]]
    },
    "Extract Video URL": {
      "main": [
        [{"node": "Upload to YouTube Shorts", "type": "main", "index": 0}],
        [{"node": "Facebook Reel — Start Upload", "type": "main", "index": 0}],
        [{"node": "Upload to TikTok", "type": "main", "index": 0}]
      ]
    },
    "Upload to YouTube Shorts": {
      "main": [[{"node": "Compile Publish Results", "type": "main", "index": 0}]]
    },
    "Facebook Reel — Publish": {
      "main": [[{"node": "Compile Publish Results", "type": "main", "index": 1}]]
    },
    "Upload to TikTok": {
      "main": [[{"node": "Compile Publish Results", "type": "main", "index": 2}]]
    },
    "Compile Publish Results": {
      "main": [[{"node": "Log to Google Sheets", "type": "main", "index": 0}]]
    }
  },
  "settings": {
    "executionOrder": "v1",
    "saveManualExecutions": true,
    "errorWorkflow": ""
  },
  "meta": {
    "instanceId": "faceless-finance-channel-v1"
  }
}
```

**Step 2: Verify JSON is valid**

Run: `python3 -c "import json; json.load(open('/c/Users/intim/Desktop/faceless-channel/n8n/workflows/faceless-finance-daily.json')); print('Valid JSON')"`
Expected: `Valid JSON`

---

## Task 6: n8n Variables Configuration

**Files:**
- Create: `n8n/README.md`

**Step 1: Create n8n README with variables to set**

Create `C:\Users\intim\Desktop\faceless-channel\n8n\README.md`:

```markdown
# n8n Workflow Setup

## Import Workflow
1. Open n8n → Workflows → Import → Select `workflows/faceless-finance-daily.json`

## Required n8n Variables
Go to: n8n Settings → Variables → Create each:

| Variable Name | Value | Where to get it |
|---|---|---|
| OPENAI_API_KEY | sk-... | platform.openai.com/api-keys |
| ELEVENLABS_API_KEY | ... | elevenlabs.io → Settings |
| ELEVENLABS_VOICE_ID | ... | elevenlabs.io → Voice Library |
| PEXELS_API_KEY | ... | pexels.com/api |
| CREATOMATE_API_KEY | ... | creatomate.com → Settings |
| CREATOMATE_TEMPLATE_ID | ... | After creating template in Creatomate |
| FACEBOOK_PAGE_ID | ... | Facebook Page → About |
| FACEBOOK_ACCESS_TOKEN | ... | Facebook Developer Portal |
| TIKTOK_ACCESS_TOKEN | ... | TikTok Developer Portal |
| GOOGLE_SHEET_ID | ... | From Google Sheets URL |

## Required n8n Credentials
Create these in n8n Settings → Credentials:

1. **YouTube OAuth2 API** → type: YouTube OAuth2
   - Client ID + Secret from Google Cloud Console
   - Click "Connect" to authorize

2. **Google Sheets OAuth2 API** → type: Google Sheets OAuth2
   - Same Google Cloud project
   - Click "Connect" to authorize

## Activating
1. Open the workflow
2. Click the Activate toggle (top right)
3. Workflow runs daily at 9 AM

## Manual Test Run
Click "Execute Workflow" to trigger manually
Watch the execution in the workflow view
Check Google Sheets log after completion (~3 minutes total)

## Troubleshooting
- **ElevenLabs 401**: Check API key and voice ID are correct
- **Pexels no results**: Try a broader query in the topic generator prompt
- **Creatomate render failed**: Check template ID matches your created template
- **YouTube 403**: Re-authorize YouTube OAuth2 credential
- **Facebook 190**: Access token expired — generate a new long-lived token
```

---

## Task 7: Verification Checklist

**Step 1: Create verification checklist**

Create `C:\Users\intim\Desktop\faceless-channel\sheets\verification-checklist.md`:

```markdown
# Pre-Launch Verification Checklist

## API Keys (check each)
- [ ] OpenAI API key works: curl -H "Authorization: Bearer $OPENAI_API_KEY" https://api.openai.com/v1/models
- [ ] ElevenLabs voice ID copied correctly from voice library
- [ ] Pexels API key tested: curl -H "Authorization: REDACTED" "https://api.pexels.com/videos/search?query=money&per_page=1"
- [ ] Creatomate API key valid and template ID set
- [ ] YouTube OAuth2 connected and authorized in n8n
- [ ] Facebook long-lived token not expired (check in Graph API Explorer)
- [ ] TikTok access token valid

## n8n Variables Set
- [ ] All 10 variables added to n8n Settings → Variables
- [ ] Both OAuth2 credentials created and authorized

## Google Sheets
- [ ] Sheet created with correct headers
- [ ] Sheet ID copied correctly from URL
- [ ] Google Sheets credential authorized in n8n

## Creatomate Template
- [ ] Template uploaded at creatomate.com
- [ ] Template ID copied to n8n variable
- [ ] Test render done manually to verify layout

## First Run
- [ ] Run workflow manually (Execute Workflow button)
- [ ] Watch execution — no red error nodes
- [ ] Video file appears in Creatomate renders dashboard
- [ ] YouTube Shorts tab shows new video
- [ ] Facebook Reels shows new reel
- [ ] TikTok draft or published post visible
- [ ] Google Sheets row added with all URLs

## Final Step
- [ ] Activate workflow toggle (runs daily 9 AM from now on)
```

**Step 2: Verify all files exist**

Run: `find /c/Users/intim/Desktop/faceless-channel -type f | sort`

Expected output:
```
.env.example
SETUP.md
creatomate/finance-shorts-template.json
docs/plans/2026-03-05-faceless-finance-channel-design.md
docs/plans/2026-03-05-faceless-finance-channel.md
n8n/README.md
n8n/workflows/faceless-finance-daily.json
prompts/script-writer.md
prompts/topic-generator.md
sheets/setup.md
sheets/verification-checklist.md
```

---

## Cost Summary

| Tool | Monthly Cost | Notes |
|---|---|---|
| n8n (self-hosted) | Free | Already running |
| OpenAI GPT-4o | ~$5-10 | ~30 videos/month, ~2K tokens each |
| ElevenLabs Starter | $5 | 10K chars/month = 30+ scripts |
| Pexels API | Free | Unlimited |
| Creatomate | $29 | 100 renders/month = 3+ months buffer |
| YouTube/FB/TikTok APIs | Free | All free with account |
| transfer.sh | Free | Temp file hosting, files expire 14d |
| Google Sheets | Free | |
| **Total** | **~$39-44/mo** | Under $50 budget |

---

## Monetization Timeline

| Month | Milestone | Action |
|---|---|---|
| 1-3 | Build library (90+ videos) | Just let it run |
| 3-4 | 500+ subscribers | Apply for affiliate programs (Robinhood, Acorns) |
| 4-6 | 1K subs + 10M Shorts views | Apply for YouTube Partner Program |
| 6+ | YPP active | Finance CPM $10-25 = $1K-2.5K/mo at 100K views |
| Ongoing | Scale | Add more channels in other niches |
