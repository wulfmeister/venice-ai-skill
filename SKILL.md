---
name: venice-api
description: Build AI apps with Venice API - privacy-first, uncensored, OpenAI-compatible. Covers chat, images, audio, video, embeddings, web search.
version: 1.0.0
author: jirikitariki
repository: https://github.com/jirikitariki/venice-ai-skill
keywords:
  - venice-ai
  - api
  - openai-compatible
  - privacy
  - ai
  - image-generation
  - text-to-speech
  - video-generation
  - embeddings
license: MIT
---

# Venice API

Privacy-first AI platform with OpenAI SDK compatibility. Zero data retention.

## Base Config

```
Base URL: https://api.venice.ai/api/v1
Auth: Bearer token (API key from venice.ai/settings/api)
```

## Venice-Specific Parameters

Use `extra_body` with `venice_parameters`:

| Parameter | Type | Description |
|-----------|------|-------------|
| `enable_web_search` | string | "off", "on", "auto" |
| `enable_web_scraping` | boolean | Scrape URLs in message (max 3) |
| `enable_web_citations` | boolean | [REF]0[/REF] format citations |
| `strip_thinking_response` | boolean | Show/hide think blocks |
| `disable_thinking` | boolean | Disable reasoning |
| `character_slug` | string | Use AI character |
| `include_venice_system_prompt` | boolean | Disable Venice defaults |

## Key Endpoints

```
POST /chat/completions     # Text + vision + audio + video input
POST /image/generate       # Image generation
POST /image/upscale       # Upscale images (<25MB)
POST /image/edit          # Inpaint/edit
POST /audio/speech        # TTS (60+ voices)
POST /audio/transcriptions # Transcription (Beta)
POST /video/quote         # Get price estimate
POST /video/queue         # Queue video generation
POST /video/retrieve      # Poll for completion
POST /video/complete      # Delete from storage
POST /embeddings          # Vector embeddings
GET  /characters/{slug}  # Get character info
GET  /models              # List models with capabilities
GET  /billing/balance    # Check USD/DIEM balance
```

## Rate Limits

| Tier | Requests/min | Tokens/min |
|------|-------------|------------|
| XS   | 500         | 1M         |
| S    | 75          | 750K       |
| M    | 50          | 750K       |
| L    | 20          | 500K       |

Image: 20/min | Audio: 60/min | Video queue: 40/min

## Video Workflow (Async)

1. Quote: `POST /video/quote` → get price
2. Queue: `POST /video/queue` → get queue_id
3. Poll: `POST /video/retrieve` with queue_id
   - Returns JSON while PROCESSING
   - Returns video/mp4 when done
4. Complete: `POST /video/complete` → delete storage

## Pricing

- Text: Per 1M tokens (check model pricing)
- Web Search: $10/1K calls
- Web Scraping: $10/1K calls (up to 3 URLs)
- TTS: $3.50/1M characters
- Transcription: $0.0001/audio second

See: https://docs.venice.ai/overview/pricing

## Model Selection

| Use Case | Model |
|----------|-------|
| Fast chat | `qwen3-4b` |
| General | `llama-3.3-70b` |
| Reasoning | `deepseek-ai-DeepSeek-R1` |
| Code | `qwen3-coder-480b-a35b-instruct` |
| Uncensored | `venice-uncensored` |

Get all models: `GET /models` (check `capabilities` for vision, function calling, web search)

## Privacy Tiers

- **Private**: Zero data retention
- **Anonymized**: Not affiliated with user
- **Uncensored**: No content filtering

## Gotchas

1. API key shown once only - copy immediately
2. Image/audio URLs must be publicly accessible
3. Video is async - must poll, not instant
4. Venice appends system prompts (disable with `include_venice_system_prompt: false`)
5. Rate limits are per-key, not per-account

## Error Codes

- 401: Invalid API key
- 402: Insufficient balance (check `/billing/balance`)
- 429: Rate limit (check `x-ratelimit-*` headers)
- 422: Content policy violation (try uncensored model)

## Response Headers

Key headers to check:
- `CF-RAY` - Request ID for support
- `x-ratelimit-remaining-requests`
- `x-venice-balance-usd`
- `x-venice-model-deprecation-warning`

## Resources

- Docs: https://docs.venice.ai
- Models: https://docs.venice.ai/overview/models
- Pricing: https://docs.venice.ai/overview/pricing
- Keys: https://venice.ai/settings/api
