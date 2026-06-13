# The Knowledge Base

The thing in the middle of Kipple Index. Everything else reads from it or writes to it.

Companion article: **#005 — A Knowledge Graph You'd Actually Talk To**.

## Why a graph, not a chat history

Most assistants remember by holding a transcript open and hoping the relevant lines are near the top. This works for the duration of a conversation and almost never afterwards. The graph approach is the opposite: the things being remembered are addressable nouns and verbs, not paragraphs. *Who* was at the gig last Friday is a different question from *what* was discussed about the gig in October, and the data model knows this.

Every fact accumulated by the system is annotated with what it's about and where it came from. The store is a single SQLite file. There is no fashionable graph database, no embedding service, no vector index. SQLite is boring on purpose; boring software fails less.

## The four shapes

| Shape | What it is | Example |
|---|---|---|
| **Entity** | A noun. Person, place, device, project, event, organisation. | *William Bickmore* (a great-grandfather, served in the Royal Field Artillery in WWI). |
| **Relationship** | A directed verb between two entities, optionally bracketed by dates. | *William Bickmore* `worked_at` *Royal Field Artillery* (1915–1918). |
| **Attribute** | A property of one entity — the columns where columns make sense. | The Weald & Downland Museum has a latitude and a longitude. |
| **Fact** | Prose with provenance. The narrative layer. | A short paragraph saying when, where, and from whom a particular detail was learned. |

Relationships and attributes are where the schema lives. Facts are where the texture lives. The two layers serve different purposes — the schema is what the consumers query against; the facts are what the system reads back to itself when it needs context.

## Categories that earn their keep

Every fact has a category. The list is short and pragmatic — added to only when an existing category genuinely cannot hold the new thing:

- `biography` — durable life facts about people
- `architecture` — design decisions about the system itself
- `troubleshooting` — symptom-to-fix bridges
- `hardware_quirk` — device gotchas
- `logbook` — session work that produced a code change
- `journal` — daily narrative
- `purchase` — orders, suppliers, prices

The categories matter because they drive retrieval. A search for "why did the kettle drip last winter" pulls from `troubleshooting`. A search for what a great-grandfather did during the war pulls from `biography`. Mixing the two would make both searches worse.

## Provenance and audit

Each fact carries the source it came from (`source`), an anchor within that source (`source_anchor`), a confidence level (0.0 to 1.0), and an importance band. Every update to a fact is recorded in an audit trail by a SQL trigger — there's no way to silently rewrite history. When a fact is wrong, the rule is to *supersede* it with a corrected version rather than delete the original. The old fact stays in the record with a lower importance; the new fact carries the truth.

This sounds bureaucratic. It is. Bureaucracy is also the only thing that has reliably stopped the system from quietly lying to itself.

## What it isn't

- **Not a CRM.** There are no campaigns, no leads, no funnels. People are recorded because they're part of the texture of a life, not because they need cultivating.
- **Not a chatbot memory buffer.** The graph survives between conversations, between weeks, between machine reboots. A conversation is one of the things that writes to it; it is not the only thing.
- **Not a SaaS knowledge base.** There is no search ranking optimisation, no "this article was helpful" widget. It optimises for one reader, who is paying attention.
- **Not exotic.** SQLite. Python. A single file you could fit on a USB stick.

The whole architecture is a year of "what's the simplest thing that would have stopped me forgetting that?" iterated about a thousand times.

---

*See also: [`00-overview.md`](./00-overview.md), [`../ARTICLE_INDEX.md`](../ARTICLE_INDEX.md).*
