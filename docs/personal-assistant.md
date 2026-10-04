# Personal Assistant (Companion)

A private, memory-rich personal companion — Dot/Muse-style, but **self-hosted and only for the owner**. Built on [OpenClaw](https://docs.openclaw.ai), reachable on WhatsApp, powered by Claude (`cloud-smart`).

See the [companion diagram](architecture.md#personal-assistant).

---

## What it is

- A **dedicated OpenClaw agent** (`companion`), separate from the coding `main` agent.
- **WhatsApp is routed to it** — your phone talks to the companion; coding happens in your CLI/IDE agents.
- **Actively engaged**: proactive morning + evening check-ins, follows up on open threads, learns you over time.
- **Model:** `cloud-smart` (Claude Sonnet) for genuine warmth, nuance, and reliable memory/tool use.

---

## How it works

| Piece | Where | Role |
|---|---|---|
| Agent | `companion` (OpenClaw) | Isolated agent, model `local/cloud-smart` |
| Gateway | `ai.openclaw.gateway` :18789 | Always-on; routes WhatsApp → companion |
| Workspace | `~/.openclaw/workspace-companion/` | Persona + memory files |
| Channel | WhatsApp (allowlisted number) | Text + voice (Whisper transcription) |

### Persona & memory (files loaded every session)

| File | Purpose |
|---|---|
| `SOUL.md` | Personality — warm, honest, low-friction, proactive |
| `USER.md` | Your profile; seeded with observed preferences, grows as it learns you |
| `MEMORY.md` | Curated long-term memory (main/private sessions only) |
| `memory/YYYY-MM-DD.md` | Daily raw notes |

Because memory is **file-based**, even scheduled/isolated runs stay in-character and context-aware — no live conversation thread required.

---

## Proactivity (scheduled)

Cron jobs run *as the companion* and deliver to WhatsApp:

| Job | Schedule | What |
|---|---|---|
| `companion-morning` | 7:30 AM daily | Warm good-morning, asks your focus, follows up on open threads |
| `companion-evening` | 9:00 PM daily | Wind-down, how the day went, captures what's worth remembering |
| `morning-briefing` | 8:00 AM daily | (separate) AI/tech briefing with web search |

```bash
openclaw cron list                        # see jobs
openclaw cron run <job-id>                # fire one now (sends a real WhatsApp)
openclaw cron edit --id <id> --cron "..."  # change schedule
```

---

## Using it

Just message it on WhatsApp like a person. It will:
- Ask your name and invite you to name *it* on first contact.
- Remember what matters and bring it forward naturally.
- Be direct and honest (no sycophancy) — tuned to how the owner likes to be dealt with.

Test from the CLI without WhatsApp:
```bash
openclaw agent --agent companion --message "hey" --thinking off --json
```

---

## Cost & privacy

- **Cost:** ~16k input tokens/message on `cloud-smart` (persona + bundled skills + tool schemas) ≈ **$0.05/msg**. A chatty day can reach a few dollars. Mitigations: trim unused skills (big win), or set a hard cap at [console.anthropic.com](https://console.anthropic.com) → Billing.
- **Privacy:** the companion runs locally, but because it uses a cloud model, **conversation content goes to Anthropic.** For fully private operation, switch it to a local model (`local/local-coder`) — at the cost of warmth/memory quality. This is the core trade-off of a 16 GB machine.

---

## Roadmap ideas (V2+)

- **Trim skills** on the companion to cut per-message cost and latency.
- **Richer memory** — structured profile + better recall (the real Dot/Muse "magic").
- **Voice replies** (TTS) to complement Whisper input.
- **Event-driven follow-ups** via OpenClaw flows (not just time-based crons).
