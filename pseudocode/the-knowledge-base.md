# The Knowledge Base — pseudocode

The core of a personal knowledge graph: how a fact is matched to a node before it is
allowed in, how a relationship is checked for direction, and how a wrong answer is
superseded rather than erased. Four primitives only: entities (nodes), relationships
(directed lines), attributes (labels on a node), facts (stated truths with provenance).

## 1. Insert a fact, with provenance (and idempotence)

```
function insert_fact(content, category, source, source_anchor,
                     importance="medium", confidence=0.9, context=""):
    # Unique key is (source, source_anchor, category): re-running the same
    # insert is a no-op rather than a duplicate.
    row = INSERT OR IGNORE INTO facts
            (content, context, category, source, source_anchor,
             importance, confidence, created_at = now())
    fact_id = row.id or lookup_existing(source, source_anchor, category)
    return fact_id
```

## 2. The vacuum-cleaner guard: a fact must know its node

```
function ingest_batch(candidate_facts):
    # Source documents are raw material, not facts. Each extracted fact must
    # attach to a specific entity, or it does not get in.
    if len(candidate_facts) > SMALL_BATCH_LIMIT:
        for f in candidate_facts:
            if f.link_entity is None:
                reject(batch, reason="unanchored bulk fact: which entity? "
                                     "what relationship? (this is not a vacuum cleaner)")
                return

    for f in candidate_facts:
        entity = resolve_or_create_entity(f.subject_text)   # dedup-guarded, below
        insert_fact_with_link(f.content, f.category, f.source, f.anchor,
                              subject_id=entity.id,
                              predicate=f.predicate,
                              object_id=f.object_id)
```

## 3. Fact + relationship, each committed independently

```
function insert_fact_with_link(content, category, source, anchor,
                               subject_id, predicate, object_id):
    fact_id = insert_fact(content, category, source, anchor)   # commits on its own
    try:
        insert_relationship(subject_id, predicate, object_id, source=source)
    except Exception as e:
        # The fact is already durable; never lose it because a link failed.
        log_warning("fact", fact_id, "saved but relationship failed:", e)
    return fact_id
```

## 4. Directional read-back before a line is drawn

```
function insert_relationship(subject_id, predicate, object_id,
                             valid_from=None, valid_to=None, source=None):
    assert predicate in PREDICATE_REGISTRY        # controlled vocabulary only

    # Lint: refuse a line that points the wrong way or duplicates an existing
    # edge under a synonym predicate.
    if reversed_direction_likely(subject_id, predicate, object_id):
        raise LintError("read this aloud: does '" + describe(subject_id) + " "
                        + predicate + " " + describe(object_id) + "' point the right way?")
    for existing in relationships_between(subject_id, object_id):
        if same_synonym_group(existing.predicate, predicate):
            raise LintError("already linked as '" + existing.predicate
                            + "'; do not add near-synonym '" + predicate + "'")

    INSERT INTO relationships
        (subject_id, predicate, object_id, valid_from, valid_to,
         source, created_at = now())
```

## 5. A detail is a label on a node, not a sentence to search

```
function get_detail(entity, key):
    # Walk to the node and read the label. Do NOT grep the fact text.
    if entity.attributes has key:
        return entity.attributes[key]
    return NOT_ON_FILE        # only after the node itself was checked
```

## 6. Supersede, never overwrite

```
function correct_fact(old_fact_id, new_content):
    # The past is not edited. Mark the old belief, write the new one beside it.
    UPDATE facts SET status = "superseded" WHERE id = old_fact_id   # audit-logged
    new_id = insert_fact(new_content, ...,
                         context = "supersedes #" + old_fact_id)
    return new_id            # both rows survive; the change of mind is part of the record
```

## 7. Dedup: generate candidates, never auto-merge

```
function resolve_or_create_entity(name_text):
    candidates = find_existing(name_text)     # normalised-equality, token-subset,
                                              # shared-tokens, fuzzy-ratio
    if candidates is empty:
        return create_entity(name_text)

    best = strongest(candidates)
    if best.confidence == HIGH:
        return best                           # silently reuse the node
    else:
        enqueue_for_human_review(name_text, candidates)
        return create_entity(name_text)       # provisional; review may merge later
    # A missed duplicate is recoverable; a wrong merge mis-attaches facts forever,
    # so an uncertain match is a question for a person, not an automatic merge.
```
