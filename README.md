# Open Yodel Protocol Specification

The **Yodel protocol** is the open communication protocol behind [Open Yodel](https://github.com/openyodel). It standardizes how any client (smartphone, browser, car, smart speaker, dedicated hardware) talks to any AI backend (Ollama, LiteLLM, vLLM, Claude API, OpenAI API, custom server) — via voice or text, in real time.

> *Yodel — calling across distance, catching the echo.*

## Versions

| Version | Status | Transport | Spec |
|---------|--------|-----------|------|
| **v1** | Draft | HTTPS + SSE | [spec](v1/spec.md) \| [OpenAPI](v1/openapi.yaml) |
| v2 | Planned | + WebSocket (bidirectional) | — |

## Design Philosophy

Yodel v1 builds on the OpenAI-compatible `/v1/chat/completions` endpoint and extends it with **optional** headers and metadata. A backend that doesn't know Yodel still works — every Yodel request is a valid OpenAI-compatible request. Extensions are additive, never breaking.

## Quick Start

Send a standard chat completion request with optional Yodel headers:

```http
POST /v1/chat/completions HTTP/1.1
X-Yodel-Version: 1
X-Yodel-Mode: ephemeral
X-Yodel-Input: voice
Authorization: Bearer <api-key>
Content-Type: application/json

{
  "model": "llama3",
  "stream": true,
  "messages": [
    { "role": "user", "content": "What's the weather like?" }
  ]
}
```

The response is a standard SSE stream. Yodel-aware backends may include a `yodel` event with TTS URLs and session metadata.

## Repository Structure

```
v1/
  spec.md          Human-readable protocol specification
  openapi.yaml     OpenAPI 3.1 machine-readable spec
decisions/
  ADR-NNN-*.md     Architecture Decision Records (ADRs)
```

## Related

- [openyodel/swift](https://github.com/openyodel/swift) — iOS SDK (Apple SpeechAnalyzer + WhisperKit)
- [openyodel/js](https://github.com/openyodel/js) — Web/PWA SDK (Web Speech API + Whisper WASM)

## License

This specification is published under [MIT](LICENSE).
