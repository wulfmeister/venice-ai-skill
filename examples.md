# Venice API Examples

Detailed code examples for the Venice API. See `SKILL.md` for the quick reference.

## Setup

### Python

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-api-key",
    base_url="https://api.venice.ai/api/v1"
)
```

### TypeScript

```typescript
import OpenAI from "openai";

const openai = new OpenAI({
  apiKey: process.env.VENICE_API_KEY,
  baseURL: "https://api.venice.ai/api/v1",
});
```

### Environment Variables

```bash
# .env
VENICE_API_KEY="your-api-key"
```

---

## Chat Completions

### Basic Chat

```python
response = client.chat.completions.create(
    model="venice-uncensored",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is Venice AI?"}
    ]
)

print(response.choices[0].message.content)
```

### With Web Search

```python
response = client.chat.completions.create(
    model="llama-3.3-70b",
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

### With Reasoning

```python
response = client.chat.completions.create(
    model="deepseek-ai-DeepSeek-R1",
    messages=[{"role": "user", "content": "Solve: 2 + 2 = ?"}],
    extra_body={
        "venice_parameters": {
            "strip_thinking_response": False  # Show think blocks
        }
    }
)

print(response.choices[0].message.content)
```

### Streaming

```python
stream = client.chat.completions.create(
    model="llama-3.3-70b",
    messages=[{"role": "user", "content": "Write a story"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### With Vision

```python
response = client.chat.completions.create(
    model="vision-capable-model",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "What do you see?"},
                {"type": "image_url", "image_url": {"url": "https://example.com/image.jpg"}}
            ]
        }
    ]
)
```

### With Function Calling

```python
response = client.chat.completions.create(
    model="llama-3.3-70b",
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

if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    print(tool_call.function.name, tool_call.function.arguments)
```

---

## Image Generation

### Basic Generation

```python
response = client.images.generate(
    model="z-image-turbo",
    prompt="A cute cat sitting on a windowsill",
    size="1024x1024",
    n=1
)

import base64
image_data = response.data[0].b64_json
with open("cat.png", "wb") as f:
    f.write(base64.b64decode(image_data))
```

### With Style and Options

```python
response = client.images.generate(
    model="nano-banana-pro",
    prompt="A serene canal in Venice at sunset",
    style_preset="Cinematic",
    aspect_ratio="16:9",
    resolution="2K",
    cfg_scale=7.5,
    negative_prompt="Clouds, rain, low quality",
    seed=12345,
    safe_mode=False
)
```

### Upscale

```python
response = client.images.upscale(
    model="default-upscale-model",
    image="base64-encoded-image-or-url",
    scale=4,
    enhance=True
)

with open("upscaled.png", "wb") as f:
    f.write(response.content)
```

### Edit/Inpaint

```python
response = client.images.edit(
    model="qwen-edit",
    image="base64-encoded-image",
    prompt="remove the tree",
    aspect_ratio="1:1"
)
```

---

## Text-to-Speech

```python
response = client.audio.speech.create(
    model="tts-kokoro",
    voice="af_sky",
    input="Hello! Welcome to Venice Voice.",
    response_format="mp3",
    speed=1.0
)

with open("speech.mp3", "wb") as f:
    f.write(response.content)
```

### Voice Options

Available voices: `af_alloy`, `af_aoede`, `af_bella`, `af_heart`, `af_jadzia`, `af_sky`, `am_adam`, `am_echo`, `am_michael`, `am_onyx`, `bf_alice`, `bm_daniel`, `zf_xiaoxiao`, `zm_yunxi`, and many more.

---

## Transcription

```python
with open("audio.mp3", "rb") as f:
    response = client.audio.transcriptions.create(
        model="nvidia/parakeet-tdt-0.6b-v3",
        file=("audio.mp3", f, "audio/mpeg"),
        response_format="json",
        timestamps=True,
        language="en"
    )

print(response.text)
print(response.timestamps)  # If timestamps=True
```

Supported formats: WAV, FLAC, M4A, AAC, MP4, MP3

---

## Video Generation

```python
import time
import requests

BASE_URL = "https://api.venice.ai/api/v1"
HEADERS = {"Authorization": f"Bearer {API_KEY}"}

# 1. Quote first
quote = requests.post(
    f"{BASE_URL}/video/quote",
    json={"model": "wan-2.5-preview-image-to-video", "duration": "5s"},
    headers=HEADERS
)
print(f"Price: ${quote.json()['quote']}")

# 2. Queue
queue = requests.post(
    f"{BASE_URL}/video/queue",
    json={
        "model": "wan-2.5-preview-image-to-video",
        "prompt": "A peaceful lake at sunrise",
        "duration": "5s",
        "image_url": "https://example.com/image.jpg"
    },
    headers=HEADERS
)
queue_id = queue.json()["queue_id"]

# 3. Poll for completion
while True:
    result = requests.post(
        f"{BASE_URL}/video/retrieve",
        json={"model": "wan-2.5-preview-image-to-video", "queue_id": queue_id},
        headers=HEADERS
    )
    
    if result.headers.get("content-type") == "video/mp4":
        with open("video.mp4", "wb") as f:
            f.write(result.content)
        break
    
    time.sleep(5)  # Wait before next poll

# 4. Complete (cleanup)
requests.post(
    f"{BASE_URL}/video/complete",
    json={"model": "wan-2.5-preview-image-to-video", "queue_id": queue_id},
    headers=HEADERS
)
```

---

## Embeddings

```python
response = client.embeddings.create(
    model="text-embedding-bge-m3",
    input="The quick brown fox jumped over the lazy dog"
)

embedding = response.data[0].embedding
print(f"Dimension: {len(embedding)}")
```

### Batch

```python
response = client.embeddings.create(
    model="text-embedding-bge-m3",
    input=["text1", "text2", "text3"]  # Up to 2048 items
)

for item in response.data:
    print(f"Index {item.index}: {len(item.embedding)} dims")
```

---

## Checking Balance

```python
response = requests.get(
    "https://api.venice.ai/api/v1/billing/balance",
    headers={"Authorization": f"Bearer {API_KEY}"}
)

balance = response.json()
print(f"Can consume: {balance['canConsume']}")
print(f"Currency: {balance['consumptionCurrency']}")
print(f"USD balance: {balance['balances']['usd']}")
print(f"DIEM balance: {balance['balances']['diem']}")
```

---

## Error Handling

```python
import time

def make_request_with_retry(client, request_func, max_retries=3):
    for attempt in range(max_retries):
        try:
            return request_func()
        except Exception as e:
            if attempt < max_retries - 1:
                wait_time = 2 ** attempt
                print(f"Retry in {wait_time}s...")
                time.sleep(wait_time)
            else:
                raise

# Check rate limits from response headers
def check_limits(response):
    remaining = response.headers.get("x-ratelimit-remaining-requests")
    reset = response.headers.get("x-ratelimit-reset-requests")
    print(f"Remaining: {remaining}, Resets at: {reset}")
```

---

## Migration from OpenAI

```python
# Before (OpenAI)
client = openai.OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

# After (Venice) - just change base_url
client = openai.OpenAI(
    api_key=os.getenv("VENICE_API_KEY"),
    base_url="https://api.venice.ai/api/v1"
)
```

Venice parameters like web search go in `extra_body`:

```python
response = client.chat.completions.create(
    model="venice-model",
    messages=[...],
    extra_body={
        "venice_parameters": {
            "enable_web_search": "auto"
        }
    }
)
```

---

## Resources

- API Docs: https://docs.venice.ai
- Models: https://docs.venice.ai/overview/models
- Pricing: https://docs.venice.ai/overview/pricing
- API Keys: https://venice.ai/settings/api
- Discord: https://discord.gg/askvenice
