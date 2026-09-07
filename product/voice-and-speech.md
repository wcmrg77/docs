---
title: "Voice & Speech"
description: "Configure text-to-speech and speech-to-text settings"
---

Configure how your agent sounds — the voice, speaking speed, and speech recognition.

## Text-to-Speech (TTS)

The TTS provider converts the agent's text responses into spoken audio.

### Providers

| Provider | Voices | Notes |
|----------|--------|-------|
| **Cartesia** | 10+ voices | Default. High quality, fast. |
| **ElevenLabs** | Multiple | Premium voice quality, custom voice cloning |
| **Gemini Live** | Puck, Fenrir, etc. | Only with Gemini Live LLM |
| **OpenAI Realtime** | marin, etc. | Only with OpenAI Realtime LLM |

### Voice selection

Each provider offers a set of predefined voices. Select from the dropdown in the agent detail page.

**Custom voice:** If you have a custom voice model (e.g., a cloned voice on ElevenLabs), enter the voice UUID in the custom voice ID field.

### Speaking rate

Adjust how fast the agent speaks:

| Rate | Effect |
|------|--------|
| **0.5** | Very slow |
| **1.0** | Normal speed (default) |
| **1.5** | Noticeably faster |
| **2.0** | Maximum speed |

**Note:** Speaking rate is not adjustable for Native Realtime providers (Gemini Live, OpenAI Realtime).

## Speech-to-Text (STT)

The STT provider transcribes the caller's speech into text for the LLM.

| Provider | Notes |
|----------|-------|
| **Deepgram** | Default. Fast and accurate for German and English. |
| **ElevenLabs** | Good multilingual support |
| **Mistral** | Alternative option |

**Note:** STT is not configurable for Native Realtime LLMs — they have built-in speech recognition.

## Who can edit

| Role | Voice/TTS | Speaking Rate | STT |
|------|:---------:|:------------:|:---:|
| Dev-Admin | Full | Full | Full |
| Client-Admin | Full | Full | Hidden |
| Client-Employee | Read-only | Read-only | Hidden |
