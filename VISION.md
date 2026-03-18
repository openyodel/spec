# OPEN YODEL

**A universal semantics for human-machine communication**

*Vision & Invitation — Version 0.1, March 2026*

Open Yodel is an open protocol that gives humans a universal way to talk to AI agents and smart devices — across any medium. HTTP, Bluetooth, LoRa, ultrasound, even the power grid. The protocol doesn't care how the signal travels. It defines what is called, not how the sound carries.

The name says it all: a yodeler in the Alps doesn't control who hears the call or how it echoes through the valley. He just yodels. Whoever hears it, knows what it means.

Today, every AI provider, every smart home platform, every IoT ecosystem speaks its own language. Open Yodel proposes a shared one — lightweight enough to fit in 7 bytes for discovery, rich enough for full conversations when needed.

This document is both a vision and an invitation. It describes where Open Yodel is heading, what already exists, what's missing, and how you can help build it.

MIT License — [github.com/openyodel](https://github.com/openyodel)

---

> *"I am neither the air nor the mouth nor the vocal cords nor the language. I am a yodeler. Whoever hears me, knows."*

---

## Table of Contents

1. [The Origin](#part-i--the-origin)
2. [The Layer Model: How Yodelers Find Each Other](#part-ii--the-layer-model-how-yodelers-find-each-other)
3. [Transport Agnosticism: Across Any Medium](#part-iii--transport-agnosticism-across-any-medium)
4. [Intelligent Response Routing](#part-iv--intelligent-response-routing)
5. [Architecture: Open, Modular, Community-Driven](#part-v--architecture-open-modular-community-driven)
6. [What Exists, and What Doesn't](#part-vi--what-exists-and-what-doesnt)
7. [An Invitation: Building Bridges](#part-vii--an-invitation-building-bridges)
8. [What You Can Build with Yodel](#part-viii--what-you-can-build-with-yodel)
9. [Roadmap: Where the Journey Goes](#part-ix--roadmap-where-the-journey-goes)
10. [The Story We Tell Together](#part-x--the-story-we-tell-together)

---

## Part I — The Origin

### The Problem

The world of artificial intelligence is growing explosively. Agents are becoming more powerful, more diverse, and more ubiquitous. At the same time, the landscape of communication protocols is fragmenting into ever more silos. MCP connects agents to tools. A2A connects agents to each other. But the most fundamental connection — the one between a human and an agent — has no standard.

Every provider builds its own interface. Every device speaks its own language. Every ecosystem is an island. A human who wants to communicate with an AI agent needs to know which app to open, which API to call, which protocol the device understands. It's as if you had to learn a different word for "hello" for every phone.

In parallel, the world of the Internet of Things comprises billions of devices communicating over dozens of protocols: Zigbee, Z-Wave, Matter, Bluetooth, LoRa, WiFi, MQTT, KNX, and many more. Here too: every device an island, every protocol a dialect. And neither of these worlds — the AI agent world and the IoT world — has a common language.

### The Metaphor

In the Alps, there is an ancient form of communication: yodeling. A yodeler stands on a mountain peak and calls into the valley. He doesn't control who listens. He doesn't control which medium the sound travels through — air, rock, water. He doesn't even control whether anyone answers. He yodels. And whoever hears it, knows.

A yodel is not a conversation. It is a signal with meaning. A coded call that everyone within earshot understands, regardless of where they stand or what they're doing. The yodeler doesn't define how the sound propagates. He defines what is called.

Open Yodel translates this principle into the digital world. It is not a transport protocol. It is not a mesh network. It is not an API. It is a semantics — a universal layer of meaning for communication between humans and machines, capable of riding on any transport.

### The Positioning

> *"MCP connects agents to tools. A2A connects agents to each other. Open Yodel connects humans to agents."*

But this positioning tells only half the story. The complete vision is larger:

> *"Open Yodel is a universal message semantics for human-agent communication — carry it over any transport."*

---

## Part II — The Layer Model: How Yodelers Find Each Other

At the heart of the Yodel vision is a five-layer communication model that differs fundamentally from traditional request/response architectures. It is modeled not on technical abstraction layers, but on human behavior: How do two strangers in a valley find each other and begin to collaborate?

### Layer 0 — Presence: "I'm here."

The default state of a Yodel node is not silence. The default state is yodeling. Every node pulses permanently with the absolute minimum of information: its existence. No intent, no payload, no target. Just presence.

A presence beacon is extremely lightweight — a few bytes suffice:

> `0xYO + [4-byte Node-ID] + [1-byte Capability-Flags]`

Seven bytes. That fits into any transport medium: a Bluetooth LE advertisement (31 bytes), a LoRa packet (50 bytes), an ultrasonic chirp via ggwave, an MQTT topic, an NFC tag, a PLC signal over the power grid. The cost is minimal — one watt, if you will.

Layer 0 runs permanently. Day and night. It is the background hum of the Yodel network. Nodes within range know of each other. Nothing more. But nothing less, either.

### Layer 1 — Intent: "I need something."

When a node (typically a human via a device) needs something, it escalates to Layer 1. It signals: I have a need. This signal is also minimal:

> `0xYO + [4-byte Node-ID] + [1-byte Intent-Type]`

Intent types categorize the need: I'm asking something (Query). I'm commanding something (Command). I'm informing about something (Notify). Nothing more needs to be said at this level. It's the difference between "Hey, I need help!" and the detailed description of the problem.

### Layer 2 — Capability: "Can I do that?"

Nodes that receive the intent check against their own capabilities. A smart home node that can control lights responds to a command intent. An LLM agent responds to a query intent. A pure relay node responds to nothing — but it remembers that someone is looking for something.

If no node in range can fulfill the intent, that's not an error. It's information. Both parties now know: we need a third. And when that third eventually comes within range — over whatever transport — the handoff can happen.

### Layer 3 — Handoff: "That one over there can do it."

Once a capable node is identified, a direct connection is established between the requester and the capable node. The intermediary node falls back to Layer 0. Minimal. Keeps yodeling. Its work is done.

This is where Yodel fundamentally differs from centralized architectures. There is no central intermediary that permanently sits in the path. The middleman is temporary. Once the direct connection is established, he is no longer needed.

### Layer 4 — Exchange: The actual work

Only now — at Layer 4 — is substantial data exchanged. The human formulates their question. The agent answers. The light turns on. The coffee machine starts. Only here does one need bandwidth, context, sessions, streaming.

And this is where the current Yodel specification fits seamlessly: the existing JSON-based request/response format with sessions, messages array, and streaming flags is Layer 4 — the Yodel HTTP Binding, Conversational Profile. Everything that exists today remains valid. It simply gets its proper place.

### The Core Principle: Minimal Energy, Maximum Reach

The layer model follows a single core principle: energy and complexity only scale up when they are needed. Layer 0 costs almost nothing. Each subsequent layer is only activated when the previous one yields an escalation. And after the work is done, everything falls back to Layer 0.

Store-and-forward is not a feature in this model, nor is it a special case. It is the natural law of the system. If the capable node is not currently in range, the intermediary remembers the intent. It keeps yodeling. And when the right one shows up, the handoff happens. Delayed, asynchronous, naturally.

---

## Part III — Transport Agnosticism: Across Any Medium

The most revolutionary property of Yodel is its complete independence from the transport medium. Yodel does not define how the sound propagates — only what is called. This fundamentally distinguishes Yodel from all existing protocols.

### The Spectrum of Transport Media

A Yodel beacon — seven bytes — fits into literally any communication medium known to humanity:

- **Radio:** Bluetooth LE advertisements, WiFi (mDNS, probe requests), LoRa, Zigbee, NFC, cellular, satellite. The classic transport paths of IoT.
- **Acoustic:** Ultrasound via ggwave, audible tones, infrasound. Any device with a microphone and speaker is a potential Yodel node.
- **Optical:** Infrared, visible light, QR codes on screens. Every screen is a transmitter, every camera a receiver.
- **Electrical:** Powerline Communication (PLC) over the power grid. Every outlet becomes a node. No additional wiring, no new infrastructure.
- **Network:** HTTP/HTTPS, MQTT, WebSocket, UDP, TCP, DNS TXT records. The entire existing internet infrastructure.

### Yodel Bindings

For Yodel to work across different transport media, the specification defines Yodel Bindings — rules for how a Yodel is encoded, fragmented, and transmitted on a specific transport.

A Yodel-over-HTTP binding uses JSON, headers, and streaming. A Yodel-over-BLE binding encodes the same semantics in 31 bytes. A Yodel-over-LoRa binding accounts for the 50-byte payload limit and defines fragmentation. But the meaning is identical. A Yodel is a Yodel, whether it comes over HTTP or ultrasound.

This is analogous to existing patterns: CoAP relates to HTTP the way a compact Yodel binding relates to a full Yodel HTTP request. CBOR relates to JSON the way a Yodel compact format relates to the Yodel JSON format. The semantics stay the same; only the encoding adapts to the constraints of the transport.

### Yodel Profiles

Beyond bindings, Yodel defines Profiles — predefined combinations of core semantics and extensions for specific use cases:

- **Conversational:** Sessions, streaming, context management. For dialog-based human-agent communication.
- **Command:** Fire-and-forget, no response needed. For control commands: "lights on," "temperature to 22 degrees."
- **Notification:** Brief confirmation over any channel. For acknowledgments: "ticket created," "task completed."
- **Discovery:** Broadcast with capability matching. For finding capable nodes: "Who can control lights here?"
- **Relay:** Store-and-forward, time-shifted. For asynchronous communication: the mailbox pattern.

### The Key Insight

Discovery and routing run over the dumbest available medium. The exchange runs over the best available medium. A room full of devices quietly yodeling via ultrasound. A phone that yodels "I need something" — acoustically. The ESP32 on the lamp hears it, matches the intent, responds via Bluetooth. Handoff. Light turns on. Everything falls back to Layer 0.

---

## Part IV — Intelligent Response Routing

Not every action needs a response. And not every response needs to go to the same device that sent the request. Yodel defines three response classes:

### No-Response

The command "lights on" needs no confirmation — the result is self-evident. The human sees whether the light turns on. An acknowledgment would be a waste of bandwidth and attention.

### Notification

The command "create a ticket" needs a brief confirmation: "ticket created." But this confirmation doesn't have to appear on the device from which the command was sent. It can pop up on a phone, as a push notification, as a brief vibration on a smartwatch.

### Full Response

The question "explain quantum computing" requires a device with a display or audio output. If the question came from a minimal voice device, the response may need to be routed to a different device — one that can render the complexity of the answer.

### Who Decides?

The fascinating question is: who decides which response class is appropriate and which device receives the answer? Not the sending device. Not a central router. Ideally, the agent itself — or a routing layer that the community shapes as needed. The agent knows best, after processing, whether the result requires a response, and if so, how complex it is.

This concept opens the door to an entirely new kind of human-machine interaction: the human speaks their intention, anywhere, anyhow. The network finds the right agent. The agent does the work. The answer finds the human — on whichever device currently fits best. Exactly how this routing works is an open question — and an invitation.

---

## Part V — Architecture: Open, Modular, Community-Driven

### Yodel Semantic Specification (MIT License)

The foundation is the Yodel Semantic Specification — an open, freely licensed document that defines the five layers, the semantics, the intent types, and the rules for bindings. It is transport-agnostic, encoding-agnostic, and implementation-agnostic. It is the RFC. Anyone can implement it, anyone can build on it.

The spec is structured into three tiers:

- **Yodel Core:** The five fundamental fields — Who, What, Body, Where, When. The absolute minimum that fits into seven bytes.
- **Yodel Extended:** Sessions, context, streaming, model preferences. For rich conversations between human and agent.
- **Yodel Bindings:** How a Yodel looks on a specific transport. HTTP, MQTT, BLE, LoRa — each binding defines encoding, fragmentation, and response mechanism.

### The Yodel SDK (@openyodel/sdk)

The TypeScript SDK is the bridge for developers. Published on npm as `@openyodel/sdk`, it provides a simple API for sending and receiving Yodel messages. Compliance levels 1–5 ensure that implementations are interoperable. The SDK is intentionally lean and dependency-free — getting started should take minutes, not hours.

The SDK is the core of the ecosystem: the more languages and platforms are supported, the faster the network grows. A Python SDK, a Go SDK, a Rust SDK — every implementation is a contribution that strengthens the whole. If you're looking for a place to start, this is it.

### Yodel Bridges: Building Bridges

The most powerful concept in the Yodel ecosystem is bridges — translation layers that bring existing devices and agents into the Yodel network without requiring any changes to them.

A Yodel-to-MQTT bridge takes MQTT topics and makes them addressable as Yodel endpoints. A Yodel-to-HomeAssistant bridge turns every Home Assistant device into a yodeler. A Yodel-to-OpenAI bridge makes every OpenAI-compatible agent reachable. The devices and agents keep speaking what they speak. The bridge translates. From their perspective, nothing changes. From Yodel's perspective, they're suddenly yodelers.

This pattern has proven itself many times in the open-source world. Zigbee2MQTT takes an entire Zigbee network and makes it MQTT-speaking — no device is asked. Home Assistant speaks to hundreds of protocols and unifies them under a single interface. CloudFlare pushed HTTP/3 by sitting as a translation layer in front of origin servers. The origin server still speaks HTTP/1.1 — the bridge translates.

Every bridge that someone builds expands the Yodel network by thousands of devices and agents in one stroke. Building bridges — in the literal and figurative sense — is perhaps the most impactful contribution one can make to the ecosystem.

### Agent Servers: Yodel for Every AI Provider

For every major AI provider, there can be a dedicated Yodel agent server: one for Anthropic Claude, one for OpenAI, one for Ollama, and more. Each server exposes the same Yodel endpoint but speaks natively to its respective AI backend. This makes every agent a yodeler — without the agent provider having to implement anything themselves.

This is an area where the community can start building immediately. If you want to connect a new AI provider, build a Yodel agent server for it. The pattern is simple: Yodel in, provider API out, response back as Yodel. A weekend project that measurably grows the ecosystem.

---

## Part VI — What Exists, and What Doesn't

### Related Projects

An honest analysis of the landscape shows: the individual pieces of the Yodel vision exist across various projects. The combination does not.

- **Reticulum:** A cryptography-based networking stack that works over any physical medium — LoRa, WiFi, serial, UDP. Transport-agnostic, self-configuring, decentralized. Impressive and inspiring. But: Reticulum is a pure networking stack. It transports bytes. It has no concept of intent, capability matching, or human-agent communication.

- **MeshCore:** Transport-agnostic, can run over LoRa, WiFi, serial, UDP, Bluetooth. Nodes announce their presence via periodic hello packets — similar to our Layer 0. But here too: pure networking, no semantics.

- **Meshtastic:** LoRa-based mesh network for text messaging without infrastructure. Impressive in its simplicity and community size, but limited to LoRa and texting. No AI agent concept.

- **A2A, ACP, ANP, ANS:** The entire agent protocol world is working on discovery and capability matching. There are capability descriptors, agent cards, semantic routing. But everything is HTTP/cloud-first. Not a single one of these protocols considers Bluetooth, LoRa, or ultrasound.

### The Gap

Nobody combines transport-agnostic mesh networking with AI agent semantics on the thinnest possible layer. The agent protocol people don't think about 50-byte payloads. The mesh networking people don't think about intent and capability matching for AI agents. They are two worlds that have never met.

The separation is historical, not technical. Mesh networking came from the hardware/IoT world. Agent protocols came from the cloud/enterprise world. That they will meet is inevitable. Yodel stands at the crossroads — and everyone who joins stands there too.

### Potential Synergies

Yodel doesn't need to reinvent the wheel. One particularly exciting possibility: use Reticulum as the transport layer and lay Yodel on top as the semantic layer. Reticulum handles encryption, routing, and multi-hop transport. Yodel handles meaning: intent, capability, handoff. Two projects that could complement each other perfectly — if someone builds the bridge.

---

## Part VII — An Invitation: Building Bridges

### How Yodel Can Grow

Open Yodel is an open protocol. It thrives when people use it, extend it, and spread it. The question is not whether a single company can push it through. The question is whether enough people share the vision to weave a net together that is larger than any individual.

There are historical precedents showing this works. The projects that most sustainably changed the world were never the ones with the biggest marketing budget. They were the ones with the clearest idea and the lowest barrier to entry: Linux, Git, MQTT, HTTP itself. They weren't "rolled out." They were "picked up."

### Where to Start

There are many ways to contribute to the Yodel ecosystem — from small experiments to ambitious projects:

- **Build a Yodel beacon:** An ESP32 with LoRa that yodels seven bytes into the air every few seconds. A Raspberry Pi that yodels over MQTT. A script that yodels over UDP. Every beacon increases the presence of the network.

- **Build a bridge:** Yodel-to-HomeAssistant. Yodel-to-MQTT. Yodel-to-Alexa. Every bridge brings thousands of existing devices into the Yodel network without a single one needing to be replaced.

- **Build an agent server:** A Yodel endpoint for Ollama, for OpenAI, for Mistral, for any AI provider that has an API. The pattern is simple and repeatable.

- **An SDK in a new language:** Python, Go, Rust, C, Swift — every implementation lowers the barrier for an entire developer community.

- **Specify a Yodel binding:** What does Yodel look like over BLE? Over LoRa? Over MQTT? Every binding opens a new transport channel.

- **Contribute to the spec:** The Yodel Semantic Specification is a living document. There are plenty of open questions: What exactly does capability discovery look like? How does routing over multiple hops work? How do you secure store-and-forward chains?

- **Just yodel:** Install the SDK, write three lines of code, and send your first Yodel. Sometimes the most important contribution is simply getting started.

### The Yodel Presence Network

One of the most beautiful ideas for spreading the protocol is as simple as the protocol itself: What if Yodel were already everywhere? Not as a finished product, but as a signal. A background hum waiting to be discovered.

Imagine: a few hundred nodes worldwide, doing nothing but sending the Yodel beacon on various channels. Seven bytes. On everything available. A LoRa node on a rooftop in Berlin. An MQTT topic on a public broker. A UDP packet regularly traversing the internet. An ultrasonic chirp in a makerspace.

None of these nodes do anything useful. They just yodel. Layer 0. Presence. And eventually, someone stumbles upon it. A network nerd who notices a peculiar 7-byte packet arriving on a certain port. Or a maker who spots a recurring signal on their SDR. They get curious. They search the byte sequence. They find the spec. And suddenly, they yodel back.

For this to work, two things are needed: First, the beacon must be recognizable and searchable — the magic bytes must be so unique that whoever finds them lands at the spec within 30 seconds. Second, the path from discovery to participation must take no more than ten minutes. `npm install`, three lines of code, yodel. Or better yet: a single HTML page that runs in the browser and immediately sends and receives Yodels.

### Who Needs Yodel Today

Yodel is not meant for "someday." There are communities that have the problem today:

- Home Assistant users with a hundred devices and twenty protocols, who have no unified way to let an AI agent talk to their smart home.
- Open-source AI enthusiasts running local models via Ollama or LM Studio, looking for a standardized interface for end-user devices.
- Makers and IoT tinkerers who program ESP32s, mount LoRa nodes on rooftops, and are looking for a unified protocol that works across hardware boundaries.
- People who run their own infrastructure and value digital sovereignty — who don't want their communication with AI agents routed through a cloud.
- Organizations with specialized communication needs — from emergency call systems to industrial applications — looking for an open, adaptable solution.

---

## Part VIII — What You Can Build with Yodel

An open protocol is only as valuable as what gets built on top of it. Yodel is intentionally designed not just as a communication standard, but as a platform for products, services, and projects that can build on it.

### Products and Services

When you build on Yodel, you build on a standard — not on a proprietary API that could be shut down tomorrow. This opens possibilities:

- **Managed Yodel gateways:** Hosted infrastructure for people who don't want to self-host. A service that connects devices and agents, with dashboard, logging, and monitoring.

- **Yodel-compatible hardware:** Ready-made devices — a LoRa beacon for the rooftop, a voice dongle for the desk, a relay node for the power outlet — that speak Yodel out of the box.

- **Specialized bridges:** Commercial bridges for enterprise ecosystems: Yodel-to-ServiceNow, Yodel-to-Salesforce, Yodel-to-SAP. Every company looking to integrate AI agents into existing infrastructure needs translation.

- **Industry-specific solutions:** Yodel as the communication layer for care services, logistics companies, building management. Everywhere people need to communicate with distributed systems without sitting in front of a screen.

- **Consulting and integration:** Knowing how to integrate Yodel into existing architectures is valuable. Consulting, training, custom implementations.

### Open-Source Projects

Not everything has to be a business. Some of the most exciting possibilities are pure community projects:

- **Yodel-over-Reticulum:** Perhaps the most elegant bridge: Reticulum for transport and encryption, Yodel for semantics. A mesh network where AI agents and IoT devices find each other and communicate across arbitrary physical media.

- **Yodel DevTools:** A browser-based tool that visualizes Yodel beacons, shows nodes on a map, captures and debugs messages. The "Wireshark for Yodel."

- **Yodel Playground:** An interactive web experience where people can understand and try Yodel in five minutes. Send, receive, route — all in the browser.

- **Academic research:** The layer model, the question of capability discovery across heterogeneous transport media, emergent routing — there is genuine research to be done here.

### The Bigger Picture

If Yodel gains traction as a standard, further horizons open: certification of Yodel-compatible devices, managed Yodel infrastructure for enterprises, enterprise features like single sign-on integration, a foundation to steward the protocol long-term. But these are future scenarios — they emerge organically when the foundation is solid.

The beauty of an open protocol: nobody needs to wait for permission. If you have an idea, you can start building it tomorrow. The spec is open. The SDK is available. The community is growing.

---

## Part IX — Roadmap: Where the Journey Goes

Open Yodel is a young project. Some things exist, some are vision. Here is an honest inventory — and a forward look that is intentionally framed as an invitation.

### What Exists Today

- **@openyodel/sdk:** TypeScript SDK on npm, 40 tests, compliance levels 1–5 documented. Ready for experimentation.
- **Yodel specification:** HTTP binding with JSON format, sessions, streaming. Stable enough for initial implementations.
- **openyodel GitHub organization:** Open repos, MIT license, ready for contributions.

### Next Steps — And Where Help Is Needed

- **Formalize the Yodel Semantic Specification:** The five core fields (Who, What, Body, Where, When) as a standalone document, independent of the HTTP binding. This is the most important foundational work.

- **First alternative binding:** Yodel-over-MQTT or Yodel-over-BLE as proof of concept that the semantics truly work transport-agnostically.

- **First bridge:** Yodel-to-HomeAssistant or Yodel-to-MQTT. The proof that existing devices can join the Yodel network without modification.

- **SDKs in additional languages:** Python and Go as the next priorities. Every SDK lowers the barrier for an entire developer community.

- **Agent servers for more providers:** OpenAI, Ollama, Mistral. The pattern is simple and repeatable.

- **Layer 0 prototypes:** BLE beacon, LoRa beacon, ggwave beacon, MQTT beacon. First experiments with the presence network.

### The Further Future — Shaped Together

- Capability discovery across heterogeneous transport media.
- Hop-based routing: messages that find their way through a mesh.
- Compact format for low-bandwidth media (LoRa, BLE, PLC).
- Yodel-over-Reticulum: the most elegant union of two complementary projects.
- Emergent network topologies: what happens when a thousand nodes yodel?
- Response routing: the agent decides which device gets the answer.
- Industry-specific profiles: care, logistics, smart home, industrial.

This roadmap is not a deadline list. It is a map of open possibilities. Every item is an invitation. Whoever finds one of them exciting can start tomorrow.

---

## Part X — The Story We Tell Together

Every great technology has a story that is bigger than its specification. TCP/IP was not just a protocol — it was the promise that any computer can talk to any other, regardless of the manufacturer. The World Wide Web was not just HTML — it was the promise that any person can find any information, regardless of where they are.

Open Yodel is the promise that any human can communicate with any machine, over any medium, at any time. Not through an app, not through an account, not through a cloud. But through a universal signal that is understood everywhere.

The story goes like this:

> *"Imagine you're standing on a mountain peak. Below you lies a valley full of devices — lamps, thermostats, servers, agents, cars, washing machines. Each speaks its own language. None understands the others. And you're standing up there, and all you want is for the light to turn on."*

> *"So you yodel. No cable, no pairing, no account. Just a short call that echoes through the valley. And somewhere, a device that can control lights hears you. It answers. The light turns on. You keep yodeling. Maybe tomorrow someone else hears you. Maybe on the other side of the mountain. Maybe on the other side of the world."*

That is Open Yodel. Nothing more, nothing less. An open standard. A yodel that echoes through every mountain, across every medium, to every machine.

And everyone who hears it is invited to sing along.

---

**GitHub:** [github.com/openyodel](https://github.com/openyodel)

**SDK:** `npm install @openyodel/sdk`

**License:** MIT — Free for everyone. Forever.
