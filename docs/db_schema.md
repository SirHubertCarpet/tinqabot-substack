# Schema

The actual tables behind [`kb-architecture.md`](./kb-architecture.md). Companion article: **#006 — What Counts As A Fact**.

This document describes the shape of the SQLite database, not the philosophy behind it. The why-this-not-that lives in the architecture doc; this is the *what*.

## Six tables that earn their keep

| Table | Purpose | Approximate scale |
|---|---|---|
| `entities` | The nouns | low tens of thousands |
| `relationships` | The directed verbs | high tens of thousands combined with attributes |
| `attributes` | The columns where columns make sense | the biggest table by rows |
| `enhanced_facts` | Prose with provenance | tens of thousands |
| `fact_links` | Which facts attach to which entities | thousands |
| `aliases` | Multiple names for the same noun | a thousand-odd |

There are others — `documents`, an audit table, a predicate registry, a full-text index over entity names, and a small embedding index — but the six above are the load-bearing ones.

## `entities` — the nouns

Each entity carries an `id` (integer — the only identifier the rest of the schema cares about), a `name`, a `type` (`person`, `place`, `device`, `project`, `event`, `organisation`, and a long tail), a `canonical_name` plus a `search_text` field (lowercased, used for matching and dedup), a `confidence`, a `source_anchor` recording where the entity first came from, and created/updated timestamps.

The `type` column is the one that attracts argument. The rule: a new type is added only when retrieval logic needs to *distinguish* that shape of entity from the others. Otherwise it goes in `other` and stays there.

## `relationships` — the verbs

Three columns do the work: `subject_id`, `predicate`, `object_id`. Two more give it teeth: `valid_from` and `valid_to` — ISO partial dates, both nullable. A great-grandfather served in the Royal Field Artillery between 1915 and 1918; both ends of that bracket matter, and silence on either end means *unknown*, not *forever*.

The predicate vocabulary is open but linted. Verb direction matters — a person can `worked_at` an organisation; an organisation cannot `worked_at` a person. A guard rejects inserts whose direction violates the type pair, and a registry table tracks the predicates in live use so synonyms (`worked_at` / `employed_by`) get caught before they fork the graph.

Each relationship also carries its own `source` and `source_anchor` — relationships have provenance too.

## `attributes` — where columns make sense

Some facts are columnar: a place has a latitude, a person has a date of birth, a device has a MAC address. These live in `attributes`: `entity_id`, `attribute`, `value`, `data_type`, `confidence`, and a `learned_date` — when the system was *told*, not when the value became true.

Attributes are deliberately not the same as facts. An attribute is structured retrieval (`SELECT value WHERE entity_id = X AND attribute = 'birth_date'`); a fact is something with narrative texture you'd want to read back. Some properties belong in both; most belong in one or the other. It is the biggest table in the database by a factor of three, which says something about how much of a life turns out to be key-value shaped.

## `enhanced_facts` — prose with provenance

The big one. Columns that matter:

- `content` — the prose itself
- `context` — a search-keyword line; what the writer expected future-them to grep for
- `source` — where this came from (a session name, an email identifier, a journal date)
- `source_anchor` — unique within the source; the schema enforces `(source, source_anchor, category)` uniqueness so re-runs don't duplicate
- `category` — what kind of fact this is (`biography`, `architecture`, `troubleshooting`, …)
- `importance` — `critical`, `high`, `medium`, `low` — or `superseded`, the retirement home for facts that turned out to be wrong
- `confidence` — 0.0 to 1.0; direct user statements get 0.9+, anything inferred gets less

There is deliberately **no `entity_id` on the fact table**. The link between facts and entities lives in `fact_links` — many-to-many, because a single sentence about who did what at the party touches several people, a place, and a date. Every update to a fact is captured by a SQL trigger into an audit table; history doesn't get silently rewritten.

## `aliases` — the same noun by many names

Three columns: `entity_id`, `alias`, `source`. Search resolves aliases case-insensitively before falling back to substring matching. This is what stops the system treating *Weald & Downland Living Museum* and *Weald and Downland Open Air Museum* as two different places — they're aliases of one `entity_id`, and the canonical name carries whichever form the writer prefers.

## What isn't in the schema

- **No `tags` column on facts.** Categories do the work; tags would compete with them and lose.
- **No free-text `notes` column on entities.** If it's worth recording, it's worth recording as a fact with provenance.
- **No `created_by` column anywhere.** Single-user system; the author is implicit.
- **Almost no embeddings.** The one concession is a small vector index over fact content, bolted on inside the same SQLite file as an auxiliary retrieval path. Structured queries do the heavy lifting; the embedding index is a fallback for fuzzy recall, not the spine of the system.

The schema is small enough to fit the whole picture in your head, and that was the goal. Adding a column has a high bar; adding a table has a higher one.

---

*See also: [`kb-architecture.md`](./kb-architecture.md), [`../ARTICLE_INDEX.md`](../ARTICLE_INDEX.md).*
