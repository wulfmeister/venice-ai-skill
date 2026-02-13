# Venice AI Skill for OpenCode

A comprehensive skill for building AI applications with the Venice API - a privacy-first, uncensored AI platform with zero data retention and OpenAI SDK compatibility.

## What is Venice AI?

Venice is an AI API platform offering:
- **Privacy-first** - Zero data retention
- **Uncensored models** - No content filtering
- **OpenAI compatible** - Just change the base URL
- **30+ models** - Text, image, audio, video generation
- **Web search** - Built-in with citations

## Installation

### For OpenCode

The skill is automatically loaded when you mention "venice" in your prompts.

### Manual Installation

Copy `SKILL.md` to your OpenCode skills directory:
```bash
cp SKILL.md ~/.opencode/skills/venice-api/SKILL.md
```

### Which file to use?

- **`SKILL.md`** (recommended) - Concise quick-reference (~140 lines). Covers all endpoints, parameters, and gotchas without eating too much context. This is the one most people should use.
- **`skill-verbose.md`** - Full detailed reference (~1,250 lines). Use this if you want to sacrifice more context window to give your AI assistant deeper knowledge of every parameter, pricing detail, code example, and edge case. Install it the same way:
  ```bash
  cp skill-verbose.md ~/.opencode/skills/venice-api/SKILL.md
  ```
- **`examples.md`** - Standalone copy-paste code examples for all endpoints. Useful as a separate reference.

## Quick Examples

### Chat
```
Use the venice-api skill to help me generate an image of a sunset
```

### Generate Image
```python
response = client.images.generate(
    model="z-image-turbo",
    prompt="A sunset over mountains",
    size="1024x1024"
)
```

### Text-to-Speech
```python
response = client.audio.speech.create(
    model="tts-kokoro",
    voice="af_sky",
    input="Hello, world!"
)
```

### Web Search
```python
response = client.chat.completions.create(
    model="zai-org-glm-4.7",
    messages=[{"role": "user", "content": "Latest AI news?"}],
    extra_body={
        "venice_parameters": {
            "enable_web_search": "auto",
            "enable_web_citations": True
        }
    }
)
```

## Features Covered

- Chat completions with streaming, vision, tool calling
- Image generation, upscaling, editing
- Text-to-speech with 60+ voices
- Audio transcription
- Video generation (async workflow)
- Embeddings
- Web search & scraping
- Reasoning models
- AI characters
- Prompt caching

## Documentation

- [Venice API Docs](https://docs.venice.ai)
- [Pricing](https://docs.venice.ai/overview/pricing)
- [Models](https://docs.venice.ai/overview/models)
- [API Keys](https://venice.ai/settings/api)

## License

MIT License - See LICENSE file for details.
