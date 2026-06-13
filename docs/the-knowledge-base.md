# The Knowledge Base

> How the index is structured: a personal knowledge graph, not a document store.

## What it is

The knowledge base is the shared memory layer for the whole house. Every other
subsystem (the carousel, the journal, the voice output, the assistant sessions)
queries it for context before asserting anything about a person, place, or project.
It is a single SQLite database, accessed only through a small set of Python helpers,
never by hand.

The defining decision is that it is a **graph**, not a pile of documents. A document
(a note, an email, an old post) is raw material from which facts are *extracted* and
attached to the things they concern. The document is the ore; the facts are the metal.
A search over ten thousand documents only ever finds what you already had words for.
A graph can be *walked*: you go to a node and read what is connected to it.

## The four primitives

Everything in the base is one of exactly four kinds of thing. The discipline is in
never inventing a fifth.

| Primitive | Table | Role |
|---|---|---|
| **Entity** | `entities` | A node: a person, place, project, device, concept. One row per real-world thing, regardless of how often it is mentioned. Has a `type` and a `canonical_name` used for dedup. |
| **Relationship** | `relationships` | A directed line between two entities: `subject` `predicate` `object`, with optional `valid_from` / `valid_to` so the link can be time-bounded. |
| **Attribute** | `attributes` | A key/value label stuck directly on one entity: a phone number, a birthday, pronouns, a corrected spelling. |
| **Fact** | `enhanced_facts` | A stated truth with provenance: `content`, `category`, `importance`, `confidence`, and a `source` / `source_anchor` recording where it came from. |

Reading a value off an attribute is cheaper and more reliable than searching the
fact text for it, so contact-shaped details (numbers, dates, pronouns) live as
attributes. Before the system reports a detail "not on file" it must first walk to
the entity and check its attributes.

## "This is not a vacuum cleaner"

The ingestion rule, stated bluntly: the base does not hoover up text. Every fact in a
batch must answer two questions before it is allowed in:

1. **Which entity does this fact attach to?**
2. **What relationship does it express?**

If neither can be answered, the fact is not inserted. A bulk batch of more than a
handful of facts is rejected outright unless each one names a specific existing entity
to link to. Source documents are never stored *as* facts; they are mined for facts that
are then matched to nodes (creating new nodes only when genuinely new).

## Directional read-back

A relationship is a *direction*, not just a connection. `employs` and `employed_by`
are the same edge read from opposite ends; writing it backwards makes the map lie in a
way that looks fine until it matters.

So any relationship using a directional verb is paraphrased in plain English and
confirmed before it is written. A write-time lint enforces this in code: it blocks
unknown predicates, flags reversed directions, and refuses a near-synonym of a
relationship that already exists (so an entity is not simultaneously `worked_at` and
`employed_by` the same place).

## Append-only: supersede, never overwrite

A wrong fact is **not** deleted or edited. It is marked `superseded` and a corrected
fact is written alongside it, dated later. The base is an append-only journal of record:
you add to it, you do not un-happen things. This preserves not just *what* is currently
believed but *what was believed before and when it changed*, which is the question that
eventually matters. Every update or delete is logged by an audit trigger.

## Identity safety

Names carry weight. Where a person has changed their name, the old form is retained
only as hidden graph plumbing so historical records still resolve, and it never surfaces
in any output. The chosen name is the only one ever spoken or written. Pronouns are an
attribute on the entity and the single source of truth; they are never inferred from a
name.

## Deduplication

Because a node must be unique, the base guards against accidentally creating two nodes
for the same thing. A shared matcher (normalised-name equality, token-subset, shared
tokens, and a fuzzy ratio) is the single source of truth for "do these two refer to the
same thing?" It is deliberately a candidate *generator*, not an arbiter: at insert time
a near-match is suppressed; for finding existing duplicates it feeds a human review
queue rather than auto-merging, because a missed duplicate is recoverable but a wrong
merge silently mis-attaches facts.

## Why SQLite, not a graph database

Simplicity and portability. The graph shape lives in ordinary relational tables and is
walked with plain queries. There is no separate graph engine to run, no extra moving
part to keep alive. The structure is a graph; the storage is boring on purpose.
