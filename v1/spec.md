# Yodel Protocol Specification v1

**Version:** 1.0-draft
**Status:** Draft
**Date:** 2026-03-07
**License:** MIT

---

## 1. Introduction

The Yodel protocol is the open communication protocol of [Open Yodel](https://github.com/openyodel). It defines how a client communicates with any AI backend over voice or text, in real time, without vendor lock-in.

Yodel v1 is intentionally close to the OpenAI-compatible `/v1/chat/completions` format. A backend that does not know Yodel still works — extensions are fully optional. The protocol value grows in v2 with bidirectional streaming, device mesh routing, and extended TTS control.

Giving the protocol its own name today makes the spec referenceable and reserves the space for real differentiation.

### 1.1 Scope

This specification covers **Yodel as Conversation** — the ongoing communication protocol between client and backend. **Yodel as Call** (the musical audio handshake for device-to-gateway pairing) is outside the scope of this protocol spec and defined separately.

### 1.2 Conventions

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT", and "MAY" in this document are to be interpreted as described in [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119).

---

## 2. Design Principles

1. **OpenAI-compatible baseline.** Any endpoint that speaks `/v1/chat/completions` with SSE streaming is immediately compatible.
2. **Additive, not replacing.** Yodel extends the OpenAI format with optional headers and metadata. It never modifies or removes existing fields.
3. **Transport-agnostic.** HTTPS + SSE is the v1 standard. WebSocket is planned for v2.
4. **Backend-agnostic.** Works with Ollama, LiteLLM, vLLM, Claude API, OpenAI API, agent platforms (n8n, OpenClaw), custom servers.
5. **MCP/A2A are backend concerns.** If the agent behind the endpoint uses MCP tools or A2A orchestration internally, that is opaque to the protocol.

---

## 3. Terminology

| Term | Definition |
|------|------------|
| **Client** | Any application or device that sends requests using the Yodel protocol (iOS app, PWA, hardware device, etc.). |
| **Backend** | Any server that handles chat completion requests. May or may not be Yodel-aware. |
| **Gateway** | An optional management layer between client and backend that handles device registration, agent configuration, and routing. Not required for Yodel communication. |
| **Agent** | A named configuration consisting of an endpoint, model, system prompt, session mode, TTS settings, and language. Users interact with agents, not raw endpoints. |
| **Session** | A logical conversation context. May be ephemeral (stateless) or persistent (client-managed history). |
| **Yodel-aware** | A backend or gateway that understands and processes Yodel headers and the `yodel` extension block. |

---

## 4. Protocol Layers

| Layer | Function | Format |
|-------|----------|--------|
| **Transport** | Connection between client and backend | HTTPS + SSE |
| **Message** | Structure of requests and responses | JSON, OpenAI-compatible with Yodel extensions |
| **Session** | Conversation context and state | Yodel session header + optional persistent history |

---

## 5. Transport Layer

### 5.1 Protocol

Yodel v1 uses HTTPS as the transport protocol. Clients MUST use TLS (HTTPS) for all communication.

### 5.2 Streaming

Responses are delivered as **Server-Sent Events (SSE)** per the [SSE specification](https://html.spec.whatwg.org/multipage/server-sent-events.html). The client MUST set `"stream": true` in the request body. Non-streaming responses are outside the scope of Yodel v1.

### 5.3 Content Types

- **Request:** `application/json`
- **Response:** `text/event-stream`

---

## 6. Request Format

### 6.1 Endpoint

```
POST /v1/chat/completions
```

This is the standard OpenAI-compatible endpoint. Yodel does not introduce new endpoints in v1.

### 6.2 Yodel Headers

All Yodel headers are **optional**. A request without any Yodel headers is a standard OpenAI-compatible request.

| Header | Type | Required | Description |
|--------|------|----------|-------------|
| `X-Yodel-Version` | Integer | RECOMMENDED | Protocol version. Currently `1`. Backends SHOULD use this to detect Yodel-aware clients. |
| `X-Yodel-Session` | String | No | Session identifier (UUID v4). Used for session continuity. |
| `X-Yodel-Device` | String | No | Device identifier (UUID v4). Persists across sessions. |
| `X-Yodel-Agent` | String | No | Agent slug identifying the active agent configuration. |
| `X-Yodel-Mode` | Enum | No | Session mode: `ephemeral` or `persistent`. Default: `ephemeral`. |
| `X-Yodel-Input` | Enum | No | Input source: `voice` or `text`. Informs the backend about the origin of the user message. |

**Standard headers** that MUST be included:

| Header | Description |
|--------|-------------|
| `Authorization` | `Bearer <api-key>` — authentication with the backend. |
| `Content-Type` | `application/json` |

### 6.3 Request Body

The request body follows the [OpenAI Chat Completions API](https://platform.openai.com/docs/api-reference/chat/create) format with an optional `yodel` extension block.

```json
{
  "model": "<model-id>",
  "stream": true,
  "messages": [
    { "role": "system", "content": "<agent-system-prompt>" },
    { "role": "user", "content": "<transcribed-text-or-user-input>" }
  ],
  "yodel": { ... }
}
```

**Required fields:**

| Field | Type | Description |
|-------|------|-------------|
| `model` | String | Model identifier (e.g., `llama3`, `claude-sonnet-4-20250514`, `gpt-4o`). |
| `stream` | Boolean | MUST be `true`. Yodel v1 is streaming-only. |
| `messages` | Array | Message array per OpenAI format. |

**Standard optional fields** (OpenAI-compatible):

Any field supported by the OpenAI Chat Completions API MAY be included (e.g., `temperature`, `max_tokens`, `top_p`). Yodel does not restrict or extend these.

### 6.4 The `yodel` Extension Block

The `yodel` block is an **optional** top-level field in the request body. A backend that does not know Yodel MUST ignore unknown fields per standard JSON processing — the request remains a valid OpenAI-compatible completion request.

```json
{
  "yodel": {
    "input_lang": "de",
    "tts": {
      "requested": true,
      "voice": "alloy",
      "provider": "elevenlabs",
      "format": "opus"
    },
    "device": {
      "type": "ios",
      "capabilities": ["audio_out", "haptic"]
    },
    "context": {}
  }
}
```

#### 6.4.1 `yodel.input_lang`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `input_lang` | String | No | BCP 47 language tag (e.g., `de`, `en-US`, `de-AT`). Indicates the language of the user input. Backends MAY use this to adjust response language. |

#### 6.4.2 `yodel.tts`

TTS configuration requested by the client. The backend MAY or MAY NOT honor these preferences.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `tts.requested` | Boolean | No | Whether the client requests TTS audio for the response. Default: `false`. |
| `tts.voice` | String | No | Preferred voice identifier (e.g., `alloy`, `nova`). Interpretation depends on the TTS provider. |
| `tts.provider` | String | No | Preferred TTS provider (e.g., `elevenlabs`, `edge`, `local`). Advisory — the backend decides. |
| `tts.format` | String | No | Preferred audio format: `opus`, `mp3`, `wav`, `aac`. Default: `opus`. |

#### 6.4.3 `yodel.device`

Device metadata. Allows Yodel-aware backends to adapt responses based on client capabilities.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `device.type` | String | No | Device type: `ios`, `android`, `web`, `car`, `speaker`, `terminal`, `embedded`. |
| `device.capabilities` | Array\<String\> | No | List of device capabilities. |

**Defined capabilities:**

| Capability | Description |
|------------|-------------|
| `audio_out` | Device can play audio (speaker/headphones). |
| `audio_in` | Device can capture audio (microphone). |
| `display` | Device has a visual display. |
| `haptic` | Device supports haptic feedback. |
| `camera` | Device has a camera. |

Additional capabilities MAY be defined by implementations. Unknown capabilities MUST be ignored by backends.

#### 6.4.4 `yodel.context`

Reserved for future use. Clients MAY send an empty object `{}`. Backends MUST ignore this field in v1.

---

## 7. Response Format

### 7.1 SSE Stream

The response is a standard SSE stream. Each event is a `data:` line containing a JSON object.

#### 7.1.1 Content Chunks

Standard OpenAI-compatible streaming chunks:

```
data: {"id":"chatcmpl-abc","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"content":"Hello"},"finish_reason":null}]}

data: {"id":"chatcmpl-abc","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"content":" there"},"finish_reason":null}]}

data: {"id":"chatcmpl-abc","object":"chat.completion.chunk","choices":[{"index":0,"delta":{},"finish_reason":"stop"}]}
```

Clients MUST handle standard OpenAI streaming chunk format as documented in the [OpenAI API reference](https://platform.openai.com/docs/api-reference/chat/streaming).

#### 7.1.2 Yodel Event

A Yodel-aware backend MAY send a `yodel` event before the `[DONE]` signal. This event carries Yodel-specific metadata.

```
data: {"yodel":{"tts_url":"https://example.com/audio/abc123.opus","session_id":"550e8400-e29b-41d4-a716-446655440000"}}
```

| Field | Type | Description |
|-------|------|-------------|
| `yodel.tts_url` | String (URL) | URL to the generated TTS audio file. Present only if TTS was requested and generated. The client SHOULD fetch and play this audio. |
| `yodel.session_id` | String (UUID) | Server-assigned session identifier. The client SHOULD include this as `X-Yodel-Session` in subsequent requests to maintain session continuity. |

Additional fields MAY be added in future protocol versions. Clients MUST ignore unknown fields in the `yodel` event.

If the client requested TTS (`tts.requested: true`) but the backend does not generate audio, the backend MAY either omit the `tts_url` field from the yodel event, omit the yodel event entirely, or return `tts_url` as `null`. Clients MUST treat all three cases identically: fall back to text-only mode without error.

#### 7.1.3 Stream Termination

The stream ends with the standard SSE termination signal:

```
data: [DONE]
```

### 7.2 Event Order

1. Zero or more **content chunks** (standard OpenAI format)
2. Zero or one **yodel event** (Yodel extension)
3. Exactly one **`[DONE]`** signal

---

## 8. Session Management

### 8.1 Session Modes

The session mode is configured **per agent**, not per session. The client communicates the mode via the `X-Yodel-Mode` header.

#### 8.1.1 Ephemeral Mode

Like a phone call. No history is retained.

- The client sends only the current user message (and the agent's system prompt).
- No `X-Yodel-Session` header is needed.
- Each request is independent.
- Ideal for quick, one-off questions.

```json
{
  "messages": [
    { "role": "system", "content": "You are a helpful assistant." },
    { "role": "user", "content": "What time is it in Tokyo?" }
  ]
}
```

#### 8.1.2 Persistent Mode

Like a chat. The client manages message history.

- The client maintains and sends the full conversation history with each request.
- The server MAY assign a session ID (via the `yodel` response event) for server-side context.
- The client SHOULD include the session ID in subsequent requests via `X-Yodel-Session`.

```json
{
  "messages": [
    { "role": "system", "content": "You are a project manager assistant." },
    { "role": "user", "content": "What did we discuss last?" },
    { "role": "assistant", "content": "We talked about the Q3 roadmap." },
    { "role": "user", "content": "Let's continue with that." }
  ]
}
```

### 8.2 Session Lifecycle

| Event | Behavior |
|-------|----------|
| Client sends request without `X-Yodel-Session` | Backend MAY create a new session and return `session_id` in the yodel event. |
| Client sends request with `X-Yodel-Session` | Backend SHOULD associate the request with the existing session. |
| Session expiration | Backend-defined. Sessions MAY expire after inactivity. Clients MUST handle missing sessions gracefully by starting a new conversation. |

---

## 9. Error Handling

### 9.1 HTTP Status Codes

Yodel uses standard HTTP status codes. Backends SHOULD return errors in the OpenAI error format:

```json
{
  "error": {
    "message": "Invalid API key",
    "type": "authentication_error",
    "code": "invalid_api_key"
  }
}
```

| Status | Meaning |
|--------|---------|
| `200` | Success — SSE stream follows. |
| `400` | Bad request — malformed JSON or invalid parameters. |
| `401` | Unauthorized — invalid or missing API key. |
| `403` | Forbidden — valid key but insufficient permissions. |
| `404` | Not found — endpoint or model not available. |
| `429` | Rate limited — too many requests. |
| `500` | Internal server error. |
| `503` | Service unavailable — backend temporarily down. |

### 9.2 Stream Errors

If an error occurs during streaming (after the `200` response has been sent), the backend SHOULD send an error event:

```
data: {"error":{"message":"Context length exceeded","type":"invalid_request_error","code":"context_length_exceeded"}}

data: [DONE]
```

Clients MUST handle errors both as HTTP responses (before streaming starts) and as events within the stream.

---

## 10. Security

### 10.1 Transport Security

All Yodel communication MUST use TLS 1.2 or higher. Clients MUST NOT send requests over plain HTTP.

### 10.2 Authentication

Authentication is handled via the standard `Authorization: Bearer <token>` header. Token management (issuance, rotation, scoping) is outside the scope of this specification and depends on the backend.

### 10.3 Header Confidentiality

Yodel headers (`X-Yodel-Device`, `X-Yodel-Session`, `X-Yodel-Agent`) contain identifiers that SHOULD be treated as sensitive. These headers MUST only be transmitted over encrypted connections.

### 10.4 Device Identity

Device identifiers (`X-Yodel-Device`) are client-generated UUID v4 values. They persist across sessions and identify a physical device. Clients SHOULD store device IDs securely (e.g., iOS Keychain, Android Keystore, browser `localStorage`).

---

## 11. Compatibility

### 11.1 OpenAI Compatibility Guarantee

Every valid Yodel v1 request is also a valid OpenAI-compatible chat completion request. This guarantee is maintained by:

- Using the standard `/v1/chat/completions` endpoint.
- Placing all Yodel extensions in custom `X-Yodel-*` headers (which non-Yodel backends ignore).
- Placing the `yodel` extension block as a top-level field in the body (which non-Yodel backends ignore as an unknown field).

### 11.2 Graceful Degradation

| Scenario | Behavior |
|----------|----------|
| Backend ignores `X-Yodel-*` headers | Client receives standard completion response. Works as expected. |
| Backend ignores `yodel` body block | No TTS URL, no session ID in response. Client falls back to text-only mode. |
| Backend does not send `yodel` response event | Client treats response as standard OpenAI stream. No TTS playback. |
| Client omits all Yodel extensions | Request is a plain OpenAI-compatible request. Fully functional. |

### 11.3 Version Negotiation

The client signals its protocol version via `X-Yodel-Version: 1`. A Yodel-aware backend SHOULD:

- Process the request according to the specified version.
- Ignore unknown version numbers gracefully (fall back to latest supported version).
- Include its supported version in the yodel response event if desired.

---

## 12. Versioning

### 12.1 Version Format

The Yodel protocol uses simple integer versioning: `1`, `2`, `3`, etc. Minor clarifications and backward-compatible additions do not increment the version number.

### 12.2 Breaking Changes

A new major version is required when:

- The transport mechanism changes (e.g., adding WebSocket support in v2).
- Existing header semantics are modified.
- Request or response structure changes in incompatible ways.

### 12.3 Forward Compatibility

Clients and backends MUST ignore unknown headers, unknown fields in JSON bodies, and unknown SSE events. This ensures forward compatibility as the protocol evolves.

---

## 13. Full Example

### Request

```http
POST /v1/chat/completions HTTP/1.1
Host: api.example.com
Authorization: Bearer sk-abc123
Content-Type: application/json
X-Yodel-Version: 1
X-Yodel-Device: d4f5a6b7-8c9d-4e0f-a1b2-c3d4e5f6a7b8
X-Yodel-Agent: cooking
X-Yodel-Mode: ephemeral
X-Yodel-Input: voice

{
  "model": "llama3",
  "stream": true,
  "messages": [
    {
      "role": "system",
      "content": "You are a friendly Austrian sous-chef. Answer briefly and practically."
    },
    {
      "role": "user",
      "content": "How long do I cook spaghetti al dente?"
    }
  ],
  "yodel": {
    "input_lang": "en",
    "tts": {
      "requested": true,
      "voice": "alloy",
      "format": "opus"
    },
    "device": {
      "type": "ios",
      "capabilities": ["audio_out", "audio_in", "display", "haptic"]
    }
  }
}
```

### Response

```
HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

data: {"id":"chatcmpl-xyz","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"role":"assistant"},"finish_reason":null}]}

data: {"id":"chatcmpl-xyz","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"content":"Cook"},"finish_reason":null}]}

data: {"id":"chatcmpl-xyz","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"content":" spaghetti"},"finish_reason":null}]}

data: {"id":"chatcmpl-xyz","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"content":" for 8"},"finish_reason":null}]}

data: {"id":"chatcmpl-xyz","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"content":" minutes"},"finish_reason":null}]}

data: {"id":"chatcmpl-xyz","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"content":" in salted"},"finish_reason":null}]}

data: {"id":"chatcmpl-xyz","object":"chat.completion.chunk","choices":[{"index":0,"delta":{"content":" boiling water."},"finish_reason":null}]}

data: {"id":"chatcmpl-xyz","object":"chat.completion.chunk","choices":[{"index":0,"delta":{},"finish_reason":"stop"}]}

data: {"yodel":{"tts_url":"https://tts.example.com/audio/chatcmpl-xyz.opus","session_id":"550e8400-e29b-41d4-a716-446655440000"}}

data: [DONE]
```

---

## Appendix A: Header Reference

| Header | Values | Default | Description |
|--------|--------|---------|-------------|
| `X-Yodel-Version` | Integer (`1`) | — | Protocol version |
| `X-Yodel-Session` | UUID v4 | — | Session identifier |
| `X-Yodel-Device` | UUID v4 | — | Device identifier |
| `X-Yodel-Agent` | String (slug) | — | Agent configuration identifier |
| `X-Yodel-Mode` | `ephemeral`, `persistent` | `ephemeral` | Session mode |
| `X-Yodel-Input` | `voice`, `text` | — | Input source |

## Appendix B: Yodel Request Block Schema

```
yodel
├── input_lang       String (BCP 47)
├── tts
│   ├── requested    Boolean
│   ├── voice        String
│   ├── provider     String
│   └── format       String (opus | mp3 | wav | aac)
├── device
│   ├── type         String (ios | android | web | car | speaker | terminal | embedded)
│   └── capabilities Array<String>
└── context          Object (reserved)
```

## Appendix C: Yodel Response Event Schema

```
yodel
├── tts_url          String (URL)
└── session_id       String (UUID v4)
```
