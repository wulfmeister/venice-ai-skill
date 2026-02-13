# Venice AI API Skill

A skill that teaches AI coding assistants how to use the Venice API - a privacy-first, uncensored AI platform with zero data retention and OpenAI SDK compatibility.

Works with **Claude Code**, **OpenCode**, **Cursor**, **Windsurf**, **Cline**, **Kilo Code**, and any tool that supports skill/rules files.

## What is Venice AI?

Venice is an AI API platform offering:
- **Privacy-first** - Zero data retention
- **Uncensored models** - No content filtering
- **OpenAI compatible** - Just change the base URL
- **30+ models** - Text, image, audio, video generation
- **Web search** - Built-in with citations

## Installation

Copy `SKILL.md` into your tool's skills/rules directory:

| Tool | Location |
|------|----------|
| **OpenCode** | `~/.opencode/skills/venice-api/SKILL.md` |
| **Claude Code** | `.claude/skills/venice-api.md` in your project |
| **Cursor** | `.cursor/rules/venice-api.mdc` in your project |
| **Windsurf** | `.windsurfrules` in your project |
| **Cline** | `.clinerules/venice-api.md` in your project |
| **Kilo Code** | `.kilocode/rules/venice-api.md` in your project |

```bash
# Example for OpenCode
cp SKILL.md ~/.opencode/skills/venice-api/SKILL.md
```

### Which file to use?

- **`SKILL.md`** (recommended) - Concise quick-reference (~140 lines). Covers all endpoints, parameters, and gotchas without eating too much context. This is the one most people should use.
- **`skill-verbose.md`** - Full detailed reference (~1,250 lines). Use this if you want to sacrifice more context window to give your AI assistant deeper knowledge of every parameter, pricing detail, code example, and edge case.
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
- [SkillMD Listing](https://skillmd.ai/skills/venice-ai-api-skill/)

## License

MIT License - See LICENSE file for details.
