# Pseudocode — the graph gardener

Companion to article **#009 — The Memory That Tidies Itself**.

The load-bearing idea: a nightly scanner that finds three kinds of graph rot and never repairs any of them. It opens the database read-only, emits findings as cards, and leaves every change to a separate human-approved apply step. Names and examples here are illustrative.

## Read-only by construction

The scanner cannot write, and not as a promise: the connection itself refuses writes.

```
function open_graph_readonly(path):
    return db.connect(path, mode = "ro")   # driver-level read-only; INSERT/UPDATE/DELETE rejected
```

## Patrol 1 — orphans (isolated entities)

An entity with no relationships and no attributes. Edge count, not fact count.

```
function scan_isolated(db):
    return db.query("""
        SELECT e.id, e.name
        FROM entities e
        WHERE e.name NOT LIKE 'SUPERSEDED %'          -- skip merge tombstones (edgeless on purpose)
          AND NOT EXISTS (SELECT 1 FROM relationships r
                          WHERE r.subject_id = e.id OR r.object_id = e.id)
          AND NOT EXISTS (SELECT 1 FROM attributes a
                          WHERE a.entity_id = e.id)
    """)
    # NOTE: an entity named in 600 facts can still be "isolated" if those
    # facts hang off OTHER nodes. Real-but-unwired => verdict "keep", not "remove".
```

## Patrol 2 — duplicates and missing links (co-occurrence)

Ignore names; read facts. Find entity pairs that co-occur often but have no relationship.

```
function scan_cooccurrence(db, min_facts = N, min_name_len = 5):
    names      = [e.name for e in db.entities()
                    if len(e.name) >= min_name_len and e.name not in SKIPLIST]
    pair_facts = {}                                  # frozenset(a, b) -> [fact ids]

    for fact in db.facts(importance in {high, critical}, length > 40):
        present = [n for n in names if word_boundary_match(n, fact.text)]
        for a, b in unique_pairs(present):
            pair_facts[frozenset({a, b})].append(fact.id)

    findings = []
    for pair, facts in pair_facts:
        if len(facts) >= min_facts and not relationship_exists(db, pair):
            findings.append({ pair: pair, count: len(facts), examples: facts[:5] })
    return sort_desc(findings, by = count)           # most co-mentions first
```

A relationship in **either** direction counts as already-linked, so the pair is deduped by an unordered set:

```
function relationship_exists(db, pair):
    a, b = pair
    return db.exists("relationships",
        (subject_id == a and object_id == b) or
        (subject_id == b and object_id == a))
```

Word-boundary match keeps "Tom" out of "tomorrow":

```
function word_boundary_match(name, text):
    return regex_search(r"\b" + escape(name) + r"\b", text, ignore_case = True)
```

The gardener stops here. It reports the **pair**, never a predicate or direction.

```
# DO NOT emit: "A works_at B".  The verb and its direction are a human call.
emit_card(kind = "merge_or_link", pair = pair, examples = facts)
```

## Merge evidence: identifiers beat names

When a human reviews a pair, the strongest signal is a shared hard identifier, not a similar name.

```
function merge_confidence(a, b):
    if shared_identifier(a, b):        # collar no. / serial / order ref on both records
        return "strong"                # names can differ wildly and it is still one thing
    if name_similar(a, b):
        return "weak"                  # suggestive only; never sufficient alone
    return "none"
```

## Triage — cards, one question at a time

Each finding becomes a card. The reviewer answers yes / no / skip.

```
function next_card(findings, decisions):
    undecided = [f for f in findings if f.id not in decisions]
    return best(undecided, by = (kind_precedence, signal_strength))
            # precedence: merge > alias > isolated; then co-mention / fact-hit count

function decide(card_id, verdict):                   # verdict in {yes, no, skip}
    decisions[card_id] = verdict                     # QUEUED ONLY — graph unchanged here
    save(decisions)
```

Recording a verdict clears the card from the queue but changes nothing in the graph. "no" on an orphan is reversible: apply ignores it, so nothing is ever deleted automatically.

## Apply — the only thing allowed to write

A separate, deliberate step. Backs up first, then enacts approved decisions.

```
function apply(decisions, dry_run = True):
    backup_database()                                # always, before any write
    for card_id, verdict in decisions:
        if verdict != "yes": continue
        if dry_run: log(card_id); continue           # default is dry-run
        match card.kind:
            merge:    merge_entities(winner, loser)   # re-point edges, alias the loser
            alias:    add_alias(entity, surface)
            isolated: audit_keep(entity)              # no-op for the schema
```

A single surgical merge, run by hand instead of the bulk apply:

```
function merge_entities(winner, loser):
    repoint(relationships, attributes, fact_links, from = loser, to = winner)
    add_alias(winner, loser.name)                    # keep the old name findable
    rename(loser, "SUPERSEDED -> " + winner.name)    # tombstone, now edgeless on purpose
    # touches nothing else
```

## Nightly wrapper

```
function gardener_cron():
    db       = open_graph_readonly(db_path())
    findings = scan_isolated(db) + scan_cooccurrence(db) + scan_alias_gaps(db)
    write_report(findings)
    if findings and not inbox_already_has_block():
        append_inbox_line("graph gardener: " + count(findings) + " findings to triage")
```

## The whole point

Notice what the scanner never does: it never merges, never deletes, never picks a predicate, never writes a single row. The cleverness is entirely in detection; the repair is fenced off behind a human and a deliberate apply. The fence is the feature.

---

*See also: [`../docs/the-graph-gardener.md`](../docs/the-graph-gardener.md), [`../docs/00-overview.md`](../docs/00-overview.md), [`../ARTICLE_INDEX.md`](../ARTICLE_INDEX.md).*
