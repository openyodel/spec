# Yodel Protocol Specification

## Inhalt

- `v1/spec.md` — Human-readable Protokoll-Spezifikation (v1.0-draft)
- `v1/openapi.yaml` — OpenAPI 3.1 Definition
- `decisions/` — Architecture Decision Records (ADRs)

## Regeln

- Die Spec verwendet **RFC 2119** Keywords (MUST, SHOULD, MAY etc.)
- Änderungen an der Spec müssen konsistent in `spec.md` UND `openapi.yaml` reflektiert werden
- Signifikante Design-Entscheidungen werden als ADR dokumentiert (`ADR-NNN-titel.md`)
- Die Spec ist die Single Source of Truth — alle SDKs leiten sich davon ab

## SDK-Sync Check

Bei Änderungen an Enums oder festen Werten in `spec.md` / `openapi.yaml` prüfen,
ob die SDK-Types noch übereinstimmen. Drift-anfällige Felder:

- `DeviceType` — OpenAPI §device.type ↔ JS SDK `src/types/config.ts`
- `DeviceCapability` — OpenAPI §device.capabilities ↔ JS SDK `src/types/config.ts`
- `TTSFormat` — OpenAPI §tts.format ↔ JS SDK `src/types/config.ts`
- `SessionMode` — OpenAPI §X-Yodel-Mode ↔ JS SDK `src/types/config.ts`
- `InputSource` — OpenAPI §X-Yodel-Input ↔ JS SDK `src/types/config.ts`
- `ChatMessage.role` — OpenAPI §messages.role ↔ JS SDK `src/types/protocol.ts`
- `finish_reason` — OpenAPI §finish_reason ↔ JS SDK `src/types/protocol.ts`
- `YodelErrorType` — OpenAPI §error.type ↔ JS SDK `src/types/errors.ts`

Wenn ein neues Enum oder eine neue feste Wertemenge in der Spec hinzukommt,
diese Liste hier UND in den SDK-CLAUDE.md's erweitern.

SDKs: `../openyodel-js/` (TypeScript), `../openyodel-swift/` (Swift, WIP)

## Offene Issues

Siehe GitHub Issues im [spec repo](https://github.com/openyodel/spec/issues) für offene Fragen (#5, #6, #7).
