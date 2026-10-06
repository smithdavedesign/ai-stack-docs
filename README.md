# Personal AI Platform

A self-hosted, **`$0`-by-default** local AI platform running on a single **MacBook Pro M1 Pro (16 GB)** — a unified coding stack plus a private personal assistant, with optional cloud "escape hatches" for hard tasks.

> **Status:** operational. One endpoint for every agent, local-first with capped cloud fallback, reboot-resilient.

This repository is the **documentation + future home** for the platform. Code can be added alongside these docs as the project grows (see [Repo layout](#repo-layout)).

---

## What's in it

- **Coding stack** — terminal + IDE agents backed by local models, with a free-model pool + cloud fallback ladder. → [docs/coding-stack.md](docs/coding-stack.md)
- **Personal assistant** — a private, memory-rich companion on WhatsApp (Dot/Muse-style), only for the owner, with **full agentic access** (writes code + runs commands in any repo) and **live integrations** (GitHub, Gmail, Calendar, Notes/Reminders, Notion). → [docs/personal-assistant.md](docs/personal-assistant.md)
- **One gateway** — every agent talks to a single OpenAI-compatible endpoint; routing, fallback, and context-compression happen behind it.

---

## System architecture

```mermaid
flowchart TB
    subgraph clients["Clients / Agents"]
        OC["OpenCode<br/>(terminal)"]
        AI["Aider<br/>(terminal)"]
        CT["Continue<br/>(VS Code)"]
        OH["OpenHands<br/>(autonomous, :3000, on-demand)"]
        OClaw["OpenClaw gateway<br/>(:18789) + WhatsApp"]
    end

    subgraph gateway["Gateway layer"]
        HR["Headroom :8787<br/>context compression"]
        LL["LiteLLM :4000<br/>router + fallback ladder<br/>key: sk-local-ai"]
    end

    subgraph models["Models"]
        OLL["Ollama :11434<br/>(tuned, GPU, keep-alive=∞)"]
        LC["local-coder · 7B"]
        L14["local-coder-14b"]
        LQ["local-qwen3 · 8B"]
        COR["cloud-or<br/>Nemotron 550B (free)"]
        CS["cloud-smart<br/>Claude Sonnet (paid)"]
    end

    OC --> HR
    AI --> HR
    CT --> HR
    OClaw --> HR
    OH --> HR
    HR --> LL
    LL --> OLL
    OLL --> LC & L14 & LQ
    LL -.free fallback.-> COR
    LL -.paid fallback.-> CS

    classDef local fill:#e6f4ea,stroke:#34a853;
    classDef cloud fill:#fde7e9,stroke:#ea4335;
    class OLL,LC,L14,LQ local;
    class COR,CS cloud;
```

**Request path:** `agent → Headroom (:8787) → LiteLLM (:4000) → Ollama (:11434) or cloud`
**Single endpoint:** `http://localhost:4000/v1` (or `:8787/v1` through Headroom) · **auth key:** `sk-local-ai`

---

## Components

| Layer | Component | Role | Port | Detail |
|---|---|---|---|---|
| Runtime | **Ollama** | Local model server (tuned) | `11434` | [coding-stack](docs/coding-stack.md#ollama) |
| Compression | **Headroom** | Shrinks context before the model | `8787` | [coding-stack](docs/coding-stack.md#headroom) |
| Router | **LiteLLM** | One API, routing, fallback ladder | `4000` | [coding-stack](docs/coding-stack.md#litellm) |
| Agent (CLI) | **OpenCode / Aider** | Terminal coding | — | [coding-stack](docs/coding-stack.md#agents) |
| Agent (IDE) | **Continue** | VS Code coding/chat | — | [coding-stack](docs/coding-stack.md#agents) |
| Agent (auto) | **OpenHands** | Autonomous multi-step tasks | `3000` | [coding-stack](docs/coding-stack.md#openhands) |
| Assistant | **OpenClaw** | WhatsApp companion + gateway | `18789` | [personal-assistant](docs/personal-assistant.md) |

---

## Documentation

| Doc | Contents |
|---|---|
| [Architecture](docs/architecture.md) | All diagrams: topology, request flow, routing, companion, resilience |
| [Coding stack](docs/coding-stack.md) | Ollama, Headroom, LiteLLM, models, agents, OpenHands |
| [Personal assistant](docs/personal-assistant.md) | Companion persona, memory, proactivity, channels, cost |
| [Operations](docs/operations.md) | Runbook, health checks, reboot resilience, troubleshooting |
| [Reference](docs/reference.md) | Ports, paths, models, secrets, external links |
| [Roadmap](docs/roadmap.md) | Where it's headed — horizons, status, deliberate no's |
| [Authentication](docs/authentication.md) | How each integration authenticates + staying logged in |

---

## Quick start

```bash
# health check (every layer)
curl -s localhost:4000/v1/models -H "Authorization: Bearer sk-local-ai"   # LiteLLM
curl -s localhost:8787/v1/models -H "Authorization: Bearer sk-local-ai"   # Headroom → LiteLLM
ollama ps                                                                 # UNTIL should say "Forever"

# use it
opencode                                   # terminal agent (default local-coder)
aider --model openai/cloud-smart           # one-off with the paid model
# WhatsApp: just message the companion
```

Full runbook: [docs/operations.md](docs/operations.md).

---

## Hard constraints (16 GB machine)

- 7–8B models comfortable; **14B only at 8k context**; 30B impossible.
- **Never** run a large local model *and* OpenHands at once.
- First request after idle/restart cold-prefills (~80 s once), then warm (~1–2 s).

---

## Repo layout

```
ai-stack-docs/
├── README.md            ← you are here
├── docs/                ← all documentation
│   ├── architecture.md
│   ├── coding-stack.md
│   ├── personal-assistant.md
│   ├── operations.md
│   └── reference.md
├── assets/              ← diagrams/images (if any)
└── (src/ …)             ← future code lives here
```

> The live configuration lives in `~/ai-stack/` (gitignored secrets) and `~/.openclaw/`. This repo documents and will eventually house code for the platform.
