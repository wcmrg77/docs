---
title: "LLM Settings"
description: "Choose providers, models, and temperature settings"
---

The LLM (Large Language Model) powers your agent's conversation intelligence — understanding callers and generating responses.

## Providers and models

| Provider | Model | Type | Notes |
|----------|-------|------|-------|
| **OpenAI** | GPT 4.1 | Standard | Best overall quality |
| **Azure** | GPT 4.1 | Standard | Same model, hosted on Azure |
| **OpenAI** | GPT 4.1 mini | Standard | Faster, lower cost |
| **Google** | Gemini 2.5 Flash | Standard | Fast with good quality |
| **Google** | Gemini 2.5 Fast Live | Native Realtime | Ultra-low latency, built-in voice |
| **Google** | Gemini 2.5 Fast Live Cascade | Cascade | Realtime with text fallback |
| **OpenAI** | Realtime | Native Realtime | Ultra-low latency, built-in voice |
| **OpenAI** | Realtime Cascade | Cascade | Realtime with text fallback |

### Standard vs Realtime vs Cascade

- **Standard** — Text-based LLM with separate TTS/STT. Most flexible voice options. Best for complex conversations.
- **Native Realtime** — Voice-native model with built-in speech. Lowest latency. Limited voice selection (provider's built-in voices only).
- **Cascade** — Starts with realtime for fast initial response, then falls back to text-based processing. Good balance of speed and quality.

## Temperature

Controls how creative vs deterministic the agent's responses are.

| Value | Behavior | Best for |
|-------|----------|----------|
| **0.0 - 0.3** | Very consistent, predictable | Factual Q&A, compliance-sensitive scenarios |
| **0.4 - 0.7** | Balanced (default: 0.7) | General customer service |
| **0.8 - 1.2** | More varied, creative | Casual conversations |
| **1.3 - 2.0** | Highly creative, less predictable | Not recommended for production |

## Impact on other settings

When you select a **Native Realtime** provider:
- **TTS provider** is automatically set to the realtime provider's built-in voice
- **STT provider** is set to null (built-in)
- **Voice selection** switches to the provider's available realtime voices
- **Speaking rate** is not adjustable

## Who can edit

Only the **Dev-Admin** role can configure LLM settings. Client-Admin and Client-Employee users don't see this section.
