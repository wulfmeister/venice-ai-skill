---
name: venice-api
description: Build AI applications with Venice API - a privacy-first, uncensored AI platform offering text generation, image generation, audio synthesis, video generation, and embeddings with zero data retention and OpenAI SDK compatibility. Use when integrating Venice API, configuring chat completions, generating images, using web search, or working with Venice-specific features like reasoning models and AI characters.
version: 1.0.0
author: wulfmeister
repository: https://github.com/wulfmeister/venice-ai-skill
license: MIT
compatibility: opencode
metadata:
  category: api-integration
  provider: venice-ai
  features:
    - chat-completions
    - image-generation
    - audio-synthesis
    - video-generation
    - embeddings
    - web-search
    - reasoning-models
    - function-calling
---

# Venice API Integration

Venice is a privacy-first AI API platform with zero data retention and OpenAI SDK compatibility. Simply change the base URL to start using Venice with existing OpenAI code.

## Quick Start

### Base Configuration

```
Base URL: https://api.venice.ai/api/v1
Auth: Bearer token (API key from venice.ai/settings/api)
```

### Getting an API Key

1. Go to [venice.ai/settings/api](https://venice.ai/settings/api)
2. Click "Generate New API Key"
3. Configure:
   - **Description**: Name for identification
   - **Type**: 
     - `Admin` - Can manage other keys (use for key rotation scripts)
     - `Inference-only` - Can only run models (safer for production apps)
   - **Expiration**: Optional date
   - **Consumption Limits**: Daily USD or DIEM caps to prevent runaway costs
4. **Copy immediately** - Venice only shows it once!

### Environment Setup

```bash
# .env file
VENICE_API_KEY="your-api-key-here"

# Or export directly
export VENICE_API_KEY="your-api-key-here"
```

```python
# Python
import os
client = openai.OpenAI(
    api_key=os.getenv("VENICE_API_KEY"),
    base_url="https://api.venice.ai/api/v1"
)
```

```typescript
// TypeScript
const openai = new OpenAI({
  apiKey: process.env.VENICE_API_KEY,
  baseURL: "https://api.venice.ai/api/v1",
});
```

### Language Examples

**Python (OpenAI SDK)**
```python
import openai

client = openai.OpenAI(
    api_key="your-api-key",
    base_url="https://api.venice.ai/api/v1"
)

response = client.chat.completions.create(
    model="venice-uncensored",
    messages=[{"role": "user", "content": "Hello World!"}]
)
```

**TypeScript/JavaScript**
```typescript
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.VENICE_API_KEY!,
  baseURL: "https://api.venice.ai/api/v1",
});

const completion = await openai.chat.completions.create({
  model: "venice-uncensored",
  messages: [{ role: "user", content: "Hello World!" }],
});
```

**Go**
```go
package main

import (
    "context"
    "fmt"
    openai "github.com/openai/openai-go"
)

func main() {
    client := openai.NewClient(
        openai.WithAPIKey("your-api-key"),
        openai.WithBaseURL("https://api.venice.ai/api/v1"),
    )

    resp, err := client.Chat.Completions.New(context.Background(), openai.ChatCompletionNewParams{
        Model: openai.F("venice-uncensored"),
        Messages: openai.F([]openai.ChatCompletionMessageParamUnion{
            openai.UserMessage("Hello World!"),
        }),
    })
    if err != nil {
        panic(err)
    }
    fmt.Println(resp.Choices[0].Message.Content)
}
```

**cURL**
```bash
curl https://api.venice.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $VENICE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "venice-uncensored", "messages": [{"role": "user", "content": "Hello!"}]}'
```

---

## Venice-Specific Parameters

Use `venice_parameters` object to access features not in the standard OpenAI API:

### Web Search & Scraping

```python
response = client.chat.completions.create(
    model="zai-org-glm-4.7",
    messages=[{"role": "user", "content": "Latest AI news?"}],
    extra_body={
        "venice_parameters": {
            "enable_web_search": "auto",        # "off", "on", "auto"
            "enable_web_scraping": True,        # Scrape URLs in user message
            "enable_web_citations": True,       # [REF]0[/REF] format citations
        }
    }
)
```

- **Web Scraping**: Automatically detects up to 3 URLs per message, scrapes and converts content into structured markdown, adds extracted text into model context
- Pricing: $10.00 per 1K calls (in addition to standard token pricing)

**Parsing Citations:**
When `enable_web_citations` is true, responses include markers like `[REF]0[/REF]` which map to search results. Parse these to link back to source URLs.

### Reasoning Models

```python
response = client.chat.completions.create(
    model="kimi-k2-thinking",
    messages=[{"role": "user", "content": "Solve this problem"}],
    extra_body={
        "venice_parameters": {
            "strip_thinking_response": False,   # Show <think/> blocks
            "disable_thinking": False,          # Disable reasoning entirely
        }
    }
)
```

### Custom System Prompts

```python
response = client.chat.completions.create(
    model="venice-uncensored",
    messages=[
        {"role": "system", "content": "Your custom system prompt"},
        {"role": "user", "content": "Hello"}
    ],
    extra_body={
        "venice_parameters": {
            "include_venice_system_prompt": False  # Disable Venice defaults
        }
    }
)
```

### AI Characters

```python
response = client.chat.completions.create(
    model="venice-uncensored",
    messages=[{"role": "user", "content": "Hello"}],
    extra_body={
        "venice_parameters": {
            "character_slug": "character-public-id"  # From character page
        }
    }
)
```

### Model Feature Suffixes

Append parameters to model ID as an alternative:

```bash
# Web search + citations via model suffix
curl ... -d '{"model": "model-id:enable_web_search=on&enable_web_citations=true", ...}'
```

---

## Venice Parameters Reference

| Parameter | Type | Description | Default |
|-----------|------|-------------|---------|
| `character_slug` | string | Public ID of a Venice character | - |
| `strip_thinking_response` | boolean | Strip `<think/>` blocks from reasoning models | `false` |
| `disable_thinking` | boolean | Disable reasoning on supported models | `false` |
| `enable_web_search` | string | Web search: `"off"`, `"on"`, `"auto"` | `"off"` |
| `enable_web_scraping` | boolean | Scrape URLs in user message (max 3 URLs) | `false` |
| `enable_web_citations` | boolean | Citations in `[REF]0[/REF]` format | `false` |
| `include_search_results_in_stream` | boolean | Search results in first stream chunk | `false` |
| `return_search_results_as_documents` | boolean | Results as tool call for LangChain | `false` |
| `include_venice_system_prompt` | boolean | Include Venice's system prompts | `true` |

---

## Response Headers

All responses include useful metadata headers:

### Rate Limiting
- `x-ratelimit-limit-requests` - Max requests allowed in current window
- `x-ratelimit-remaining-requests` - Requests remaining in current window
- `x-ratelimit-reset-requests` - Unix timestamp when window resets
- `x-ratelimit-limit-tokens` - Max tokens allowed per minute
- `x-ratelimit-remaining-tokens` - Tokens remaining in current minute
- `x-ratelimit-reset-tokens` - Seconds until token limit resets

### Balance Tracking
- `x-venice-balance-usd` - USD credit balance
- `x-venice-balance-diem` - DIEM token balance
- `x-venice-balance-vcu` - Venice Compute Units

### Request Info
- `CF-RAY` - Unique request ID (log for troubleshooting)
- `x-venice-model-id` - Model used for request
- `x-venice-model-name` - Friendly model name

### Deprecation Warnings
- `x-venice-model-deprecation-warning` - Warning message
- `x-venice-model-deprecation-date` - Deprecation date

### Content Safety
- `x-venice-is-blurred` - Image was blurred
- `x-venice-is-content-violation` - Content policy violation

**Example: Accessing Headers**
```javascript
const requestId = response.headers.get('CF-RAY');
const remainingTokens = response.headers.get('x-ratelimit-remaining-tokens');
const usdBalance = response.headers.get('x-venice-balance-usd');
```

---

## API Endpoints

### Chat Completions
```
POST /api/v1/chat/completions
```
Text generation with streaming, vision, tool calling, audio input, video input, and Venice parameters.

### Image Generation
```
POST /api/v1/image/generate
```
Generate images from text prompts.

**Request:**
```python
{
    "model": "z-image-turbo",
    "prompt": "A beautiful sunset over mountains",
    "cfg_scale": 7.5,           # CFG scale (0-20)
    "negative_prompt": "Clouds, rain",
    "width": 1024,              # Must be divisible by model's widthHeightDivisor
    "height": 1024,
    "steps": 25,               # Inference steps
    "seed": 12345,             # For reproducible results
    "variants": 1,              # Number of images (1-4)
    "style_preset": "Cinematic",
    "aspect_ratio": "16:9",     # Or "1:1", "9:16", etc.
    "resolution": "1K",         # "1K", "2K", "4K" (model dependent)
    "safe_mode": false,         # Disable content filtering
    "hide_watermark": true,
    "return_binary": false,     # true for raw image response
    "format": "webp"           # webp, png, jpeg
}
```

**Response:**
```json
{
    "id": "generate-image-1234567890",
    "images": ["base64..."],
    "timing": {"total": 5000, "inferenceDuration": 3000},
    "request": {...}
}
```

### Image Upscaling
```
POST /api/v1/image/upscale
```
Enhance and upscale existing images. File size must be < 25MB.

**Request:**
```python
{
    "model": "default-upscale-model",
    "image": "base64-encoded-image",  # or URL, or multipart file
    "scale": 4,                        # 2 or 4
    "enhance": true                    # Apply enhancement
}
```

### Image Editing
```
POST /api/v1/image/edit
```
AI-powered inpainting and image editing. File size must be < 25MB.

**Request:**
```python
{
    "model": "qwen-edit",
    "image": "base64-encoded-image",
    "prompt": "remove the tree",        # Short, descriptive
    "aspect_ratio": "1:1"
}
```

### Image Background Remove
```
POST /api/v1/image/remove-background
```
Remove background from images.

### Image Styles
```
GET /api/v1/image/styles
```
List available style presets.

**Response:**
```json
{"data": ["3D Model", "Analog Film", "Anime", "Cinematic", "Comic Book"]}
```

### Text-to-Speech
```
POST /api/v1/audio/speech
```
Convert text to audio with 60+ multilingual voices.

**Request:**
```python
{
    "model": "tts-kokoro",
    "input": "Hello, welcome to Venice Voice.",
    "voice": "af_sky",               # See voice list below
    "response_format": "mp3",         # mp3, opus, aac, flac, wav, pcm
    "speed": 1.0,                    # 0.25 to 4.0
    "streaming": false
}
```

**Voice Options (60+ voices):**
```
# Female: af_* (adult female)
af_alloy, af_aoede, af_bella, af_heart, af_jadzia, af_jessica, af_kore, 
af_nicole, af_nova, af_river, af_sarah, af_sky

# Male: am_* (adult male)  
am_adam, am_echo, am_eric, am_fenrir, am_liam, am_michael, am_onyx, 
am_puck, am_santa

# Female British: bf_*
bf_alice, bf_emma, bf_lily

# Male British: bm_*
bm_daniel, bm_fable, bm_george, bm_lewis

# Chinese: zf_*, zm_*
zf_xiaobei, zf_xiaoni, zf_xiaoxiao, zf_xiaoyi
zm_yunjian, zm_yunxi, zm_yunxia, zm_yunyang

# And more: French, Hindi, Indonesian, Japanese, Spanish...
```

### Audio Transcription (Beta)
```
POST /api/v1/audio/transcriptions
```
Transcribe audio files to text.

**Request (multipart/form-data):**
- `file`: Audio file (WAV, WAVE, FLAC, M4A, AAC, MP4, MP3)
- `model`: `nvidia/parakeet-tdt-0.6b-v3` (default) or `openai/whisper-large-v3`
- `response_format`: `json` (default) or `text`
- `timestamps`: boolean - include word-level timestamps
- `language`: ISO 639-1 code (optional, auto-detected if omitted)

**Response:**
- `text`: Transcribed text
- `duration`: Audio duration in seconds
- `timestamps`: Word/segment timestamps (if requested)

### Video Generation
```
POST /api/v1/video/queue
POST /api/v1/video/retrieve
POST /api/v1/video/complete
POST /api/v1/video/quote
```

**Workflow:**
1. **Quote**: Call `/video/quote` to get price estimate
2. **Queue**: Call `/video/queue` with model, prompt, duration, resolution, aspect_ratio, audio settings
   - Returns `model` + `queue_id`
   - Prompt max: 2500 characters
   - Duration: `5s` or `10s`
   - Resolution: `1080p`, `720p`, `480p`
   - Supports image-to-video, text-to-video
   - Audio input: WAV/MP3, max 30s, max 15MB
3. **Poll**: Call `/video/retrieve` with `model` + `queue_id`
   - Returns JSON status (`PROCESSING`) while running with timing estimates
   - Returns video/mp4 binary when complete
   - Use `delete_media_on_completion: true` to auto-delete after download
4. **Complete**: Call `/video/complete` to delete stored media after successful download

**Video Polling Example:**
```python
import time

def generate_video(client, model, prompt, duration="5s"):
    # Quote first
    quote = client.post("/video/quote", json={"model": model, "duration": duration})
    
    # Queue the job
    result = client.post("/video/queue", json={
        "model": model,
        "prompt": prompt,
        "duration": duration,
        "image_url": "https://example.com/image.jpg"
    })
    queue_id = result.json()["queue_id"]
    
    # Poll until complete
    while True:
        response = client.post("/video/retrieve", json={
            "model": model,
            "queue_id": queue_id
        })
        
        if response.headers.get("content-type") == "video/mp4":
            # Save video
            with open("video.mp4", "wb") as f:
                f.write(response.content)
            break
        
        time.sleep(5)  # Wait before next poll
```

### Embeddings
```
POST /api/v1/embeddings
```
Generate vector embeddings for semantic search.

**Request:**
```python
{
    "model": "text-embedding-bge-m3",
    "input": "The quick brown fox jumped over the lazy dog",
    "encoding_format": "float",    # or "base64"
    "dimensions": 1024              # optional, reduces dimensions
}
```

**Response:**
```json
{
    "data": [{
        "embedding": [0.002, -0.009, 0.015, ...],
        "index": 0,
        "object": "embedding"
    }],
    "model": "text-embedding-bge-m3",
    "usage": {"prompt_tokens": 8, "total_tokens": 8}
}
```

**Batch Embedding:**
```python
{
    "input": ["text1", "text2", "text3"],  # Up to 2048 items
    "model": "text-embedding-bge-m3"
}
```

### Characters API
```
GET /api/v1/characters/{slug}
```
Get character details by slug.

**Response:**
```json
{
    "data": {
        "slug": "alan-watts",
        "name": "Alan Watts",
        "description": "British philosopher...",
        "modelId": "venice-uncensored",
        "webEnabled": true,
        "adult": false,
        "photoUrl": "https://..."
    },
    "object": "character"
}
```

### API Keys Management
```
GET /api/v1/api_keys
POST /api/v1/api_keys
DELETE /api/v1/api_keys/{id}
```

**Create Key:**
```python
{
    "name": "my-api-key",
    "permissions": ["chat", "image", "audio", "video"]  # optional
}
```

### Rate Limits & Balance
```
GET /api/v1/api_keys/rate_limits
GET /api/v1/api_keys/rate_limit_logs
GET /api/v1/billing/balance
```

### Model Discovery
```
GET /api/v1/models
GET /api/v1/models/traits
GET /api/v1/models/compatibility_mapping
```
- `traits`: Returns model traits (e.g., `default`, `fastest`)
- `compatibility_mapping`: Maps OpenAI models to Venice equivalents (e.g., `gpt-4o` → `zai-org-glm-4.7`)

---

## Rate Limiting

### Text Model Tiers

| Tier | Requests/min | Tokens/min | Example Models |
|------|-------------|------------|----------------|
| XS   | 500         | 1,000,000  | `qwen3-4b`, `llama-3.2-3b` |
| S    | 75          | 750,000    | `mistral-31-24b`, `venice-uncensored` |
| M    | 50          | 750,000    | `llama-3.3-70b`, `qwen3-next-80b`, `google-gemma-3-27b-it` |
| L    | 20          | 500,000    | `kimi-k2-thinking`, `grok-41-fast`, `gemini-3-pro-preview`, `zai-org-glm-4.7` |

### Other Model Types

| Type | Requests/min |
|------|-------------|
| Image | 20 |
| Audio | 60 |
| Embedding | 500 |
| Video (queue) | 40 |
| Video (retrieve) | 120 |

### Partner Tier (Higher Limits)

| Tier | Requests/min | Tokens/min |
|------|-------------|------------|
| XS   | 500         | 2,000,000  |
| S    | 150         | 1,500,000  |
| M    | 100         | 1,500,000  |
| L    | 60          | 1,000,000  |

### Abuse Protection
- If >20 failed requests in 30 seconds → 30-second block

### Checking Your Limits

```bash
# View your rate limits
curl https://api.venice.ai/api/v1/api_keys/rate_limits \
  -H "Authorization: Bearer $VENICE_API_KEY"

# Check rate limit logs
curl https://api.venice.ai/api/v1/api_keys/rate_limit_logs \
  -H "Authorization: Bearer $VENICE_API_KEY"

# Check balance
curl https://api.venice.ai/api/v1/billing/balance \
  -H "Authorization: Bearer $VENICE_API_KEY"
```

---

## Context & File Limits

### Context Windows
- Context length varies by model (33K to 1M+ tokens)
- Check `GET /api/v1/models` → `model_spec.availableContextTokens`
- Extended context pricing (>200K tokens) applies to some models

### File Size Limits

| Endpoint | Limit |
|----------|-------|
| Image upload (generate/upscale/edit) | < 25MB |
| Audio input (TTS, transcription) | < 15MB |
| Audio transcription duration | max 30 seconds |
| Video input (video-to-video) | MP4, MOV, WebM |
| Video audio background | WAV/MP3, max 30s, max 15MB |
| Image edit pixel dimensions | Min 65536px, max 33,177,600px |

### Document Upload
- No generic document upload endpoint exists
- Use vision models for image-based document analysis
- Web scraping extracts content from URLs into context

### Image URL Requirements
- URLs must be publicly accessible (no auth required)
- Supported: http://, https://, data: URIs
- Vision models require minimum 64 pixel images

---

## Pricing Structure

Full pricing: https://docs.venice.ai/overview/pricing

### Text Models (per 1M tokens)
- **Input** - Prompt tokens
- **Output** - Completion tokens
- **Cache Read** - Discounted rate for cached prompts
- **Cache Write** - Premium rate for cache creation (some providers)
- **Extended Context** - Higher rates for >200K token inputs

### Image Generation
- Per image or per resolution tier (1K, 2K, 4K)
- Private models from $0.01/image, Anonymized from $0.04-$0.23/image

### Image Upscaling
- 2x: $0.02
- 4x: $0.08

### Image Editing
- Per edit operation: $0.02 - $0.23

### Audio
- **Text-to-Speech**: $3.50 per 1M characters
- **Speech-to-Text (Transcription)**: $0.0001 per audio second

### Video
- Pricing varies by model, duration, resolution
- Use `/video/quote` endpoint for exact quotes

### Web Search & Scraping
- **Web Search**: $10.00 per 1K calls
- **Web Scraping**: $10.00 per 1K calls
- Additional to standard token pricing

### Payment Options
- **USD** - Credit card, crypto (credits never expire)
- **DIEM** - Stake for daily compute ($1 = 1 DIEM/day)
- **Pro** - $18/month, includes one-time $10 API credit on upgrade

### Cost Estimation
```python
# Check balance before expensive operations
balance = client.get("/billing/balance")
if not balance.json()["canConsume"]:
    print("Insufficient balance!")

# Quote video before generating
quote = client.post("/video/quote", json={
    "model": "wan-2.5-preview-image-to-video",
    "duration": "5s",
    "resolution": "720p"
})
print(f"Estimated cost: ${quote.json()['quote']}")
```

---

## Model Selection Guide

### By Use Case

| Use Case | Recommended Model |
|----------|------------------|
| Fast chat/classification | `qwen3-4b` (XS tier, Private) |
| General purpose | `zai-org-glm-4.7`, `minimax-m21` |
| Complex reasoning | `kimi-k2-thinking`, `deepseek-v3.2` |
| Code generation | `grok-code-fast-1`, `qwen3-coder-480b-a35b-instruct` |
| Vision/analysis | `gemini-3-pro-preview`, `grok-41-fast` |
| No content filtering | `venice-uncensored` (Private) |
| Frontier | `openai-gpt-52`, `claude-opus-45` |
| Budget vision | `google-gemma-3-27b-it` (Private) |
| Web search | Models with `supportsWebSearch: true` |
| Function calling | Models with `supportsFunctionCalling: true` |

### By Trait

```bash
# Get auto-select models
curl https://api.venice.ai/api/v1/models/traits

# Response:
# {
#   "default": "zai-org-glm-4.7",
#   "fastest": "qwen3-4b",
#   "default_code": "qwen3-coder-480b-a35b-instruct",
#   ...
# }
```

---

## Model Capabilities

### Check Capabilities via API

```bash
curl https://api.venice.ai/api/v1/models?type=text \
  -H "Authorization: Bearer $VENICE_API_KEY"
```

Each model returns `model_spec.capabilities`:
- `supportsFunctionCalling`
- `supportsReasoning` (think blocks)
- `supportsVision`
- `supportsWebSearch`
- `supportsResponseSchema`
- `optimizedForCode`

### Privacy Tiers
- **Private** - Zero data retention
- **Anonymized** - Requests not affiliated with user
- **Uncensored** - No content filtering

### Traits
Use traits for auto-selection instead of hardcoded model IDs:
```bash
curl https://api.venice.ai/api/v1/models/traits \
  -H "Authorization: Bearer $VENICE_API_KEY"
# Returns: {"default": "zai-org-glm-4.7", "fastest": "qwen3-4b", ...}
```

---

## Prompt Caching

Venice supports automatic prompt caching on select models:

### Automatic Caching
- System prompts cached automatically on supported models
- No code changes required

### Manual Cache Control
```python
response = client.chat.completions.create(
    model="zai-org-glm-4.7",
    messages=[
        {
            "role": "system",
            "content": [
                {"type": "text", "text": "Long system prompt...", "cache_control": {"type": "ephemeral"}}
            ]
        }
    ],
    extra_body={
        "prompt_cache_key": "session-123"  # Routing hint for better cache hits
    }
)
```

### Cache Key Benefits
- Routes requests to same backend infrastructure
- Improves cache hit rates across multi-turn conversations

---

## Vision Processing

Use vision-capable models for image analysis:

```python
response = client.chat.completions.create(
    model="gemini-3-pro-preview",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "What do you see in this image?"},
                {"type": "image_url", "image_url": {"url": "data:image/jpeg;base64,..."}}
            ]
        }
    ]
)
```

### Image Input Methods
- **Base64 data URI**: `data:image/jpeg;base64,{base64_data}`
- **URL**: `https://example.com/image.jpg` (must be publicly accessible)

### Multiple Images
```python
{
    "role": "user",
    "content": [
        {"type": "text", "text": "Compare these two images:"},
        {"type": "image_url", "image_url": {"url": "https://example.com/img1.jpg"}},
        {"type": "image_url", "image_url": {"url": "https://example.com/img2.jpg"}}
    ]
}
```

### Video Input
Vision models also support video input:
```python
{
    "role": "user",
    "content": [
        {"type": "text", "text": "Describe this video:"},
        {"type": "video_url", "video_url": {"url": "https://example.com/video.mp4"}}
    ]
}
```
Supported formats: mp4, mpeg, mov, webm

---

## Audio Input

Send audio directly to vision-capable models for analysis:

```python
{
    "type": "input_audio",
    "input_audio": {
        "data": "<base64-encoded-audio>",
        "format": "wav"  # wav, mp3, aiff, aac, ogg, flac, m4a, pcm16, pcm24
    }
}
```

---

## Tool Calling / Function Calling

Define tools for model to call external APIs:

```python
response = client.chat.completions.create(
    model="grok-41-fast",
    messages=[{"role": "user", "content": "What's the weather in NYC?"}],
    tools=[
        {
            "type": "function",
            "function": {
                "name": "get_weather",
                "description": "Get current weather for a location",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "location": {"type": "string", "description": "City name"}
                    },
                    "required": ["location"]
                }
            }
        }
    ]
)

# Check if model wants to call a tool
if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    function_name = tool_call.function.name
    arguments = json.loads(tool_call.function.arguments)
```

---

## Structured Responses / JSON Mode

Force JSON responses with guaranteed schemas:

```python
response = client.chat.completions.create(
    model="zai-org-glm-4.7",
    messages=[{"role": "user", "content": "List 3 fruits with colors"}],
    response_format={"type": "json_object"}
)
```

---

## Streaming

Enable streaming for real-time responses:

```python
stream = client.chat.completions.create(
    model="zai-org-glm-4.7",
    messages=[{"role": "user", "content": "Write a story"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

**Streaming with Web Search:**
- Web search works with streaming when `include_search_results_in_stream: true`
- Search results appear in the first chunk

---

## Error Handling

### Common Error Codes
- `400` - Invalid request parameters
- `401` - Invalid API key
- `402` - Insufficient balance (check via `/billing/balance`)
- `404` - Media not found (video retrieval)
- `413` - Request payload too large
- `415` - Invalid content-type
- `422` - Content policy violation
- `429` - Rate limit exceeded (check headers)
- `500` - Server error (retry with backoff)
- `503` - Service at capacity
- `504` - Request timeout (use streaming)

### Balance Check Before Requests
```python
def can_make_request():
    balance = client.get("/billing/balance")
    return balance.json()["canConsume"]

if not can_make_request():
    # Pre-check failed - don't make the request
    print("Insufficient balance")
```

### Best Practices
1. Monitor `x-ratelimit-remaining-*` headers
2. Implement exponential backoff on 429 errors
3. Log `CF-RAY` header for support requests
4. Check balance via `/billing/balance` before expensive operations
5. Check for `x-venice-model-deprecation-warning` headers

### Content Policy Differences
- **Uncensored models**: No content filtering
- **Private/Anonymized models**: May block explicit, harmful, or policy-violating content
- Check `x-venice-is-content-violation` header on image responses

---

## Differences from OpenAI API

1. **`venice_parameters`** - Venice-specific features
2. **System Prompts** - Venice appends defaults (disable with `include_venice_system_prompt: false`)
3. **Model Ecosystem** - Use Venice model IDs, not OpenAI mappings
4. **Response Headers** - Balance tracking, deprecation warnings, content safety flags
5. **Content Policies** - More permissive with uncensored models

---

## Integrations

Venice integrates with popular frameworks:

- **LangChain** - Use `return_search_results_as_documents` for LangChain consumption
- **Vercel AI SDK** - Use Venice as provider
- **CrewAI** - Use Venice for agent workflows
- **Claude Code** - Use with Venice API
- **OpenClaw** - Build autonomous agents
- **Postman** - Import ready-to-use collection

---

## Complete Working Examples

### Simple Chat

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-api-key",
    base_url="https://api.venice.ai/api/v1"
)

response = client.chat.completions.create(
    model="venice-uncensored",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is Venice AI?"}
    ]
)

print(response.choices[0].message.content)
```

### Chat with Web Search

```python
response = client.chat.completions.create(
    model="zai-org-glm-4.7",
    messages=[{"role": "user", "content": "What are the latest AI news headlines?"}],
    extra_body={
        "venice_parameters": {
            "enable_web_search": "auto",
            "enable_web_citations": True
        }
    }
)

print(response.choices[0].message.content)
# Output may contain [REF]0[/REF] markers linking to sources
```

### Generate Image

```python
response = client.images.generate(
    model="z-image-turbo",
    prompt="A cute cat sitting on a windowsill, sunset",
    size="1024x1024",
    n=1
)

import base64
image_data = response.data[0].b64_json
with open("cat.png", "wb") as f:
    f.write(base64.b64decode(image_data))
```

### Text-to-Speech

```python
response = client.audio.speech.create(
    model="tts-kokoro",
    voice="af_sky",
    input="Hello! This is a test of the Venice text to speech system.",
    response_format="mp3"
)

with open("speech.mp3", "wb") as f:
    f.write(response.content)
```

### Transcription

```python
with open("audio.mp3", "rb") as f:
    response = client.audio.transcriptions.create(
        model="nvidia/parakeet-tdt-0.6b-v3",
        file=("audio.mp3", f, "audio/mpeg"),
        response_format="json",
        timestamps=True
    )

print(response.text)
print(response.timestamps)  # If timestamps=True
```

### Create Embeddings

```python
response = client.embeddings.create(
    model="text-embedding-bge-m3",
    input="The quick brown fox jumped over the lazy dog"
)

embedding = response.data[0].embedding
print(f"Dimension: {len(embedding)}")
```

---

## Migration from OpenAI

### Quick Switch

```python
# OpenAI
client = openai.OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

# Venice (just change base_url)
client = openai.OpenAI(
    api_key=os.getenv("VENICE_API_KEY"),
    base_url="https://api.venice.ai/api/v1"
)
```

### Parameter Mapping

| OpenAI Parameter | Venice Equivalent |
|-----------------|-------------------|
| `model` | Same (use Venice model IDs) |
| `temperature` | Same |
| `max_tokens` | Same |
| `stream` | Same |
| `tools` | Same |
| `response_format` | Same |
| N/A | `venice_parameters` (Venice-specific) |

### Key Differences

1. **Model IDs**: Use Venice IDs, not OpenAI (e.g., `venice-uncensored`, not `gpt-4`)
2. **Venice params**: Use `extra_body` for Venice-specific features
3. **Headers**: Venice adds balance and deprecation headers
4. **Uncensored option**: Use `venice-uncensored` for no content filtering

---

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| `401 Invalid API key` | Wrong or expired key | Regenerate at venice.ai/settings/api |
| `402 Insufficient balance` | No credits left | Add funds or check DIEM staking |
| `429 Rate limit` | Too many requests | Implement backoff, check headers |
| `422 Content policy` | Blocked content | Try uncensored model |
| `404 Not found` | Invalid model ID | Check available models via `/models` |
| Slow responses | Large model or queue | Use smaller tier models |

### Debug Tips

```python
# Always log CF-RAY for support
print(f"Request ID: {response.headers.get('CF-RAY')}")

# Check remaining quota
print(f"Remaining requests: {response.headers.get('x-ratelimit-remaining-requests')}")
print(f"Balance: ${response.headers.get('x-venice-balance-usd')}")

# Check for deprecation warnings
if response.headers.get('x-venice-model-deprecation-warning'):
    print("WARNING: Model is deprecated!")
```

### Testing Checklist

- [ ] API key is valid (test with `/models` endpoint)
- [ ] Balance is sufficient (check `/billing/balance`)
- [ ] Model supports required feature (check capabilities)
- [ ] Image URLs are publicly accessible
- [ ] Audio files under size limits
- [ ] Using correct content-type headers

---

## Common Gotchas

1. **Venice shows API key once** - Copy immediately when creating
2. **URLs must be public** - Vision/audio inputs can't be behind auth
3. **Web search costs extra** - $10/1K calls on top of token pricing
4. **Video is async** - Must poll, not instant
5. **System prompts are appended** - Venice adds its own by default
6. **Context caching is model-specific** - Not all models support it
7. **Uncensored ≠ everything allowed** - Still has some limits
8. **Rate limits are per-key** - Different keys have separate limits

---

## Best Practices Summary

1. **API Keys** - Keep secure, rotate regularly, never expose client-side
2. **Rate Limiting** - Monitor headers, implement exponential backoff
3. **Balance** - Track USD/DIEM balances via `/billing/balance` to avoid interruptions
4. **Pre-check** - Verify balance before expensive operations (video, image batch)
5. **Caching** - Use `prompt_cache_key` for multi-turn conversations
6. **System Prompts** - Test with/without Venice defaults
7. **Logging** - Log `CF-RAY` for troubleshooting
8. **Models** - Use traits for auto-selection, check deprecation headers
9. **Web Search** - Use `"auto"` to let model decide when to search
10. **Video Workflow** - Quote → Queue → Poll → Complete
11. **Image URLs** - Must be publicly accessible, not behind auth
12. **Content Policy** - Understand differences between uncensored vs private models
13. **Test first** - Use smaller models/fewer tokens to verify before scaling

---

## Resources

- **API Docs**: https://docs.venice.ai/api-reference/api-spec
- **Models**: https://docs.venice.ai/overview/models
- **Pricing**: https://docs.venice.ai/overview/pricing
- **API Keys**: https://venice.ai/settings/api
- **LLMs.txt Index**: https://docs.venice.ai/llms.txt
- **Swagger Spec**: https://api.venice.ai/doc/api/swagger.yaml
- **Discord**: https://discord.gg/askvenice
- **GitHub Docs**: https://github.com/veniceai/api-docs

---

## When to Use This Skill

Use when:
- Integrating Venice API into applications
- Configuring chat completions with Venice-specific features
- Enabling web search, reasoning, vision, or audio capabilities
- Generating images, videos, or speech
- Working with uncensored or privacy-focused AI models
- Migrating from OpenAI to Venice
- Troubleshooting API issues
- Understanding rate limits and balance tracking
- Implementing prompt caching strategies
- Building AI agents with Venice
