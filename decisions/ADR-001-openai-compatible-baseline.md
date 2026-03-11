# ADR-001: OpenAI-kompatibles Format als Protokoll-Baseline

**Status:** Accepted
**Datum:** 2026-03-11
**Deciders:** Open Yodel Core Team

---

## Kontext

Yodel braucht ein Basisformat für die Kommunikation zwischen Client und Backend. Die zentrale Frage war: Definieren wir ein eigenes Nachrichtenformat — oder bauen wir auf einem bestehenden Standard auf?

Die Anforderungen waren klar:

- Sofortige Kompatibilität mit möglichst vielen Backends (Ollama, LiteLLM, vLLM, OpenAI API, Claude API, …)
- Kein Vendor Lock-in für Clients oder Backends
- Streaming als First-Class Citizen
- Erweiterbar für Yodel-spezifische Features (TTS, Session, Device)
- Einfach adoptierbar — für Backend-Betreiber wie für SDK-Entwickler

---

## Entscheidung

Yodel v1 verwendet das **OpenAI Chat Completions API-Format** (`POST /v1/chat/completions`, SSE-Streaming) als unveränderliche Basis und erweitert es mit **optionalen** `X-Yodel-*`-Headern und einem optionalen `yodel`-Block im Request-Body.

Jeder valide Yodel v1 Request ist gleichzeitig ein valider OpenAI-kompatibler Request. Diese Garantie ist nicht verhandelbar.

Diese Entscheidung gilt für v1. Sie ist ein Bootstrapping-Entscheid, kein Bekenntnis zum OpenAI-Format als dauerhafter Basis. Ob v2 diese Garantie beibehält oder ein eigenes Nachrichtenformat einführt, ist eine offene Frage — dokumentiert als Action Item und als Voraussetzung für den v2-ADR.

Das Nachrichtenformat ist dabei bewusst von den definierenden Merkmalen des Protokolls getrennt. Was Yodel zu einem eigenständigen Protokoll macht, ist unabhängig davon, welches Basisformat auf der Leitung verwendet wird: Device-Identität (`X-Yodel-Device`), Input-Semantik (`X-Yodel-Input: voice | text`), Session-Modus als First-Class-Konzept (`X-Yodel-Mode`), TTS-Koordination im Stream, Discovery-Hierarchie und das clientseitig verwaltete Session-Modell. Diese Konzepte existieren als optionaler Overlay über dem Basisformat – sie sind nicht vom OpenAI-Schema abhängig und könnten auf einem anderen Basisformat gleichermaßen funktionieren.

---

## Betrachtete Optionen

### Option A: Eigenes Yodel-Format (neuer Endpoint)

Ein eigenes Nachrichtenformat unter `/v1/yodel/chat` mit eigenem Schema — optimiert für Voice, TTS und Device-Kontext.

| Dimension | Bewertung |
|-----------|-----------|
| Ausdrucksstärke | Hoch — kein Kompromiss mit OpenAI-Constraints |
| Ökosystem-Kompatibilität | Keine — jedes Backend braucht nativen Yodel-Support |
| Adoption-Hürde | Sehr hoch — Backend-Betreiber müssen aktiv implementieren |
| Erweiterbarkeit | Maximal |

**Pro:** Volle Kontrolle über das Format. Keine Einschränkungen durch OpenAI-Designentscheidungen.
**Contra:** Kein einziges Backend wäre Day-1 kompatibel. Yodel würde als Nischenprotokoll starten, das aktive Adoption erfordert — ein klassisches Henne-Ei-Problem.

---

### Option B: OpenAI-kompatibel als Baseline (gewählt)

Yodel baut auf `/v1/chat/completions` auf. Extensions kommen via HTTP-Header (ignoriert von Nicht-Yodel-Backends) und einem optionalen Body-Feld (ignoriert als unbekanntes JSON-Feld).

| Dimension | Bewertung |
|-----------|-----------|
| Ausdrucksstärke | Mittel — durch OpenAI-Schemakonventionen eingeschränkt |
| Ökosystem-Kompatibilität | Sehr hoch — hunderte kompatibler Backends sofort nutzbar |
| Adoption-Hürde | Minimal — bestehende Backends brauchen keine Änderung |
| Erweiterbarkeit | Hoch — Header und Body-Extensions sind additiv |

**Pro:** Ollama, LiteLLM, vLLM, OpenAI, Claude (via Proxy), Groq, Together AI und jeder OpenAI-kompatible Endpoint funktionieren out-of-the-box – ohne dass diese Backends das Yodel-Protokoll kennen müssen. Das ist möglich, weil Yodels definierende Konzepte (Device-Identität, Session-Modus, Input-Semantik, TTS-Koordination, Discovery) vollständig im Header- und Extension-Layer leben und nicht vom Basisformat abhängen. Yodel-Mehrwert ist sofort erlebbar — kein Warten auf Ecosystem-Adoption.
**Contra:** Das OpenAI-Format und seine Konventionen (Roles, Message-Struktur, Parameter-Namen) sind nicht unter Yodels Kontrolle. Sollte OpenAI das Format inkompatibel ändern, entsteht Druck.

---

### Option C: Anthropic-natives Format als Baseline

Analog zu Option B, aber auf Basis der Anthropic Messages API (`POST /v1/messages`, `parts`-Format).

| Dimension | Bewertung |
|-----------|-----------|
| Ökosystem-Kompatibilität | Niedrig — nur Claude-kompatible Backends |
| Beziehung zum Initiator | Nähe zum Projektursprung |
| Adoptions-Hürde | Hoch — das Anthropic-Format ist weniger verbreitet |

**Contra:** Schränkt das erreichbare Backend-Ökosystem massiv ein. Widerspricht dem Kernversprechen von Vendor-Unabhängigkeit.

---

## Trade-off-Analyse

Die entscheidende Spannung ist **Kontrolle vs. Reichweite**. Ein eigenes Format maximiert die Kontrolle, minimiert aber die initiale Reichweite auf null. Das OpenAI-Format invertiert diese Relation: maximale Reichweite ab Tag eins, moderate Einschränkungen im Schema.

Für ein junges Open-Source-Protokoll ohne bestehende Nutzerbasis ist Reichweite wichtiger als Formatreinheit. Yodels Mehrwert liegt nicht im Nachrichtenformat selbst, sondern in der Schicht darüber. Was Yodel von einem einfachen OpenAI-Proxy unterscheidet: Ein Proxy hat keine Device-Identität, kein Session-Modell, keine Input-Semantik, keine Discovery-Hierarchie. Yodel definiert, dass ein Gerät eine persistente Identität hat (`X-Yodel-Device`), dass Voice und Text als Eingabequellen unterschieden werden (`X-Yodel-Input`), dass Session-Modus pro Agent konfiguriert wird (`X-Yodel-Mode`), dass TTS im Stream koordiniert wird, und dass Clients Endpoints über eine definierte Discovery-Hierarchie finden. Diese Konzepte sind protokolldefinierend – das Nachrichtenformat ist austauschbar, sie sind es nicht. Diese Schicht lässt sich als optionaler Overlay auf jedes Basisformat legen — und der OpenAI-kompatible Layer ist der mit dem größten Hebel.

Die Einschränkungen durch das OpenAI-Format (Roles `system`/`user`/`assistant`, Message-Array, Parameter-Naming) sind in der Praxis keine: Yodel v1 braucht genau diese Struktur.

---

## Konsequenzen

**Wird einfacher:**
- Jeder Backend-Betreiber kann Yodel sofort testen — kein Setup erforderlich
- SDK-Entwickler können das Yodel-Format gegen bestehende OpenAI-Testinfrastruktur validieren
- Gradual Adoption: Yodel-Headers können schrittweise hinzugefügt werden
- Der Wert von `X-Yodel-Version` als Erkennungsmerkmal für Yodel-aware Backends bleibt erhalten

**Wird schwieriger:**
- Zukünftige Protokollerweiterungen müssen mit dem OpenAI-Format kompatibel bleiben oder auf v2 warten
- Yodel hat keine Governance-Kontrolle über sein Basisformat. Falls OpenAI das `/v1/chat/completions`-Format inkompatibel ändert, entsteht Handlungsbedarf. Das ist kein theoretisches Risiko – das Anthropic `parts`-Format ist ein konkretes Beispiel dafür, dass „OpenAI-kompatibel" keine universelle Aussage ist. Spec Issue #5 adressiert dies. Die Abhängigkeit ist akzeptiert, aber nicht ignoriert.
- Body-seitige Extensions (der `yodel`-Block) müssen als unbekannte Felder toleriert werden — kein Backend darf sie ablehnen, aber das ist Standard-JSON-Verhalten

**Muss später revisited werden:**
- Ob die OpenAI-Kompatibilitätsgarantie für v2 (WebSocket) aufrechterhalten werden kann oder ob ein eigenes Transport-Format für bidirektionales Streaming sinnvoller ist
- Wie mit Backends umgegangen wird, die das Anthropic `parts`-Format oder andere Schemas nutzen (→ Spec Issue #5)

---

## Action Items

- [x] OpenAI-Kompatibilitätsgarantie in Spec §11.1 festhalten
- [x] `yodel`-Block als top-level optionales Feld spezifizieren (§6.4)
- [x] Graceful Degradation-Tabelle in Spec §11.2 dokumentieren
- [ ] Spec Issue #5 klären: Header-Body-Trennung als explizites Designprinzip in der Spec verankern
- [ ] Entscheiden, ob v2 die OpenAI-Kompatibilitätsgarantie beibehält oder aufgibt (→ eigener ADR vor v2-Spec)
