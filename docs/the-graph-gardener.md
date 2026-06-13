# The graph gardener

The companion architecture note for article **#009 — The Memory That Tidies Itself**. Pseudocode walkthrough: [`../pseudocode/the-graph-gardener.md`](../pseudocode/the-graph-gardener.md).

## What it is

The graph gardener is a read-only maintenance scanner for the house's knowledge base. The knowledge base is a graph: entities (people, places, devices, projects), facts written about them, relationships between them, and aliases that let a search find an entity by more than one name. Left alone, a long-lived graph accumulates three kinds of quiet rot: orphan entities, duplicate entities, and missing relationships. The gardener walks the graph on a nightly schedule, finds instances of each, and surfaces them for human review. It does not repair anything itself.

The whole design is precision-first and human-in-the-loop. The gardener is a witness, not a surgeon.

## The three patrols

### Orphans (isolated entities)

An entity with **zero relationships and zero attributes**: nothing in the graph ties it to anything else. The scan is a single query. Orphans are not automatically wrong; a real thing can simply be thinly recorded. But an orphan is worth a look, because it is often the wreckage of a half-entered record.

Two refinements matter. First, the test **counts edges, not facts**: an entity can be named in hundreds of facts and still register as an orphan if those facts hang off other nodes (its devices, free text) rather than off the entity itself. Such a node is real and important but badly wired, and the correct verdict is "keep" (meaning "this is legitimate, stop asking"), not "remove". Second, **completed-merge tombstones are excluded**: when two entities are merged, the loser is renamed to a `SUPERSEDED ...` marker and stripped of edges on purpose, so flagging it as an orphan would be a false positive. The scan skips those by name.

### Duplicates (co-occurrence pairs)

The same real thing recorded more than once. This is the worst rot because it is invisible from the inside: a query answers confidently about one copy while the others silently hold the rest of the truth.

The hunt deliberately ignores names, because names are how duplicates got in. Instead it reads the facts. For each substantial fact it extracts every entity named in it using **word-boundary matching** (so "Tom" inside "tomorrow" does not match), then finds pairs of entities that **co-occur in many facts but have no relationship recorded** in either direction. Those pairs are the suspects: either a genuine missing link or two records of one thing. Noise is filtered with a minimum name length and a common-word skiplist.

The hard-won lesson: the strongest evidence for a merge is a **shared identifier** (a collar number, serial, or order reference) that appears on two differently-named records, not name similarity. A four-way split of one record was collapsed only because the same identifier sat on all the fragments.

### Missing links

A subset of the co-occurrence output: two entities that clearly belong together, appear in the same fact again and again, and yet have no relationship row. The gardener reports the pair and example facts. It deliberately **does not propose the predicate or direction**, because choosing the verb and its direction is a human judgement (directional read-back before any write).

## The cardinal rule: it never writes

The gardener opens the database in **read-only mode at the driver level**, not merely by convention. It cannot insert, update, or delete against any table. Every finding is emitted as a record for review.

This is deliberate. A merge is not a small act: every fact, relationship, attribute, and alias on the loser must be re-pointed onto the winner, and a wrong direction or wrong survivor corrupts the memory in a way that looks tidy. An automated merge that is mostly right is more dangerous than none, because the wrong fraction is now wearing the costume of a clean record. So the gardener proposes; a human disposes.

## Review and apply

Findings become single-question cards in a phone-friendly triage page, one decision at a time:

| Kind     | Yes means | No means          | Source           |
|----------|-----------|-------------------|------------------|
| merge    | merge     | different things  | co-occurrence    |
| alias    | add alias | leave it          | name-surface gap |
| isolated | keep      | flag for removal  | orphan scan      |

Cards rank by kind precedence then by signal strength (co-mention count, fact-hit count). **Decisions are only ever queued, never applied inline.** A separate, deliberate apply step (run by hand, backing up the database first) is the only thing permitted to rewrite the graph. A single surgical merge can also be run directly: it re-points relationships, attributes, and fact-links onto the winner, keeps the loser's name as an alias, and touches nothing else.

## Scheduling

The scan runs once nightly in the small hours, after a sibling scanner. Output goes to a log and a one-line summary into the AI inbox so the next session sees there are findings to triage. The inbox entry is deduplicated so repeated runs in a day do not stack.

## Why it works

The value is in the restraint. The gardener is good at noticing that something is wrong and wisely refuses to decide what right looks like. The memory stays trustworthy not because the scanner is clever, but because it finds the rot, lays it out, and does not touch it.

---

*See also: [`00-overview.md`](./00-overview.md), [`the-knowledge-base.md`](./the-knowledge-base.md), [`../pseudocode/the-graph-gardener.md`](../pseudocode/the-graph-gardener.md), [`../ARTICLE_INDEX.md`](../ARTICLE_INDEX.md).*
