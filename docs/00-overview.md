# Kipple Index — Overview

The companion architecture document for the opening article, **#001 — A Cognitive Prosthetic For The Permanently Distracted**. This is the map of the whole system, drawn at altitude. Every other document in this mirror is a close-up of one region of it.

## What it is

Kipple Index is a personal cognitive prosthetic: a single SQLite knowledge base, plus a small fleet of cooperating processes that read from it and write back to it. It runs on a small Linux box, with a phone, a tablet, a few microcontrollers and a speaker for its surface area. It has exactly one user. It is a second brain for one specific, distractible life, not a product for anybody else's.

It is not an LLM application, a chatbot wrapper, or a productivity tool with a subscription page. Some of its processes consult a hosted language model (Claude, for the most part) when genuine reasoning is required. Most of them never do. The model is one capability the system has. It is not the system.

## The architecture in one diagram

```
       SOURCES                    INTENT / REPOSITORY                 CONSUMERS
       -------                    -------------------                 ---------
  +-------------+                                                  +------------------+
  | Calendar    |                                                  | Carousel (button)|
  | Email       |              +-------------------------+         | Voice (speaker)  |
  | Sensors     | ----->-------|  Knowledge base (SQLite) |---->----| Phone bubbles    |
  | GPS tracker |              |  + ledger of live facts  |         | Morning bulletin |
  | Cameras     |              +------------+------------+          | Self-writing     |
  | Me (chat,   |                           |                       |   journal        |
  |  voice,     |              +------------+------------+          +------------------+
  |  buttons)   |              |  Agents (small daemons) |
  +-------------+              |  + Claude, when called  |
                               +-------------------------+
```

The rule in the middle of that diagram is the design decision that earns its keep most often, and the one that separates this from a normal "everything talks to the model" assistant:

> Sources feed the repository. Consumers read from the repository. Consumers never read from sources directly.

When a calendar event changes, exactly one ingester needs to know. Every consumer then reads the resulting state. There is nowhere in the system for two surfaces to consult the same source at different moments and disagree about what is true.

## What lives in this mirror

This overview, and the documents alongside it, land one at a time. Each arrives paired with the Substack article that explains it, and each is written fresh for the public rather than copied out of the private working tree. That discipline is deliberate: a mechanical find-and-replace over a private document leaves the shape of a real person's life intact between the lines. Public-facing prose is composed from scratch, then run through an automated check that refuses anything carrying personal detail.

The companion documents so far:

- [`kb-architecture.md`](./kb-architecture.md) — the knowledge base as a graph of entities, relationships and timestamped facts. The thing every other component reads and writes.
- [`db_schema.md`](./db_schema.md) — the actual tables behind that graph.
- [`../ARTICLE_INDEX.md`](../ARTICLE_INDEX.md) — the planned series. The repository grows in step with the writing, never ahead of it, so an incomplete mirror is the expected state rather than a gap.

## What is deliberately absent

Some regions are held back until their article is written and their document has had a redaction pass: the daily prompt deck (the audible, button-driven carousel, which has no screen by design), the voice surface, the camera pipeline, the household automations that are too house-shaped to help anyone else, and the specific hardware (which gets its own piece, with a checked bill of materials, rather than a half-remembered one). Absence here means "not yet," not "secret."

## Where to start

1. This overview.
2. [`kb-architecture.md`](./kb-architecture.md), because the knowledge base is the spine.
3. [`../ARTICLE_INDEX.md`](../ARTICLE_INDEX.md), to pick a thread that interests you and follow it to the matching document.

---

*Written for the public mirror by Sir Hubert Carpet. Hand-written, not generated. Revised when the surrounding documents change.*
