# Dowser: pseudocode

Meaning-based recall over the knowledge base. Keyword search matches words;
Dowser matches meaning, then grades its own confidence before it is trusted.

---

## 1. Turn text into a vector

```
EMBED_DIM = 384

function embed(text):
    # the model maps text to a fixed-length vector; similar meanings land nearby
    vector = embedding_model.encode(text)
    vector = normalise_to_unit_length(vector)   # so cosine == dot product
    assert length(vector) == EMBED_DIM
    return vector
```

## 2. Build the index once, ahead of time (nightly)

```
function build_index(records, full):
    begin savepoint
    for record in records:                      # every fact and every entity
        text = embed_text_for(record)           # the fields worth searching
        hash = sha256(text)
        if not full:
            cached = meta_lookup(record.id)
            if cached.hash == hash and cached.model == MODEL_VERSION
                    and vector_exists(record.id):
                continue                         # unchanged: skip
        vector = embed(text)
        replace_vector(record.id, vector)
        upsert_meta(record.id, hash, MODEL_VERSION, now)
    release savepoint

function gc_orphans():
    # drop vectors whose source record was deleted
    for id in vector_ids_with_no_source_record():
        delete_vector(id); delete_meta(id)
```

## 3. The embed daemon (keep the model warm)

```
# The model takes ~8s to load. Load it ONCE, then serve embeds over a socket.

daemon main():
    model = load_model()                        # the one slow load, at boot
    socket = listen(SOCKET_PATH)
    loop:
        conn = socket.accept()
        request = conn.recv(timeout = 5s)
        vector = embed_in_process(request.text, model)   # NOT embed_via_daemon!
        conn.send({ embedding: vector })

# Callers prefer the warm daemon, fall back to a local load if it is down.
function embed(text):
    try:
        return embed_via_daemon(text)           # ~0.1s, model already in RAM
    except (no_socket, refused, timeout):
        return embed_in_process(text, load_model())   # ~8s safety net

# GOTCHA: the daemon's handler must call embed_in_process directly.
# If it called embed() (which prefers the socket), it would try to connect
# back to itself while busy serving, and deadlock.
```

## 4. Nearest-neighbour search

```
function fuzzy_search(query, k, min_cosine):
    if k == 0: return []
    qvec = embed(query)
    candidates = []
    for store in [fact_vectors, entity_vectors]:
        for (id, distance) in nearest(store, qvec, k):
            cosine = 1 - distance^2 / 2          # L2 distance -> cosine for unit vectors
            if cosine >= min_cosine:
                candidates.append(Candidate(id, cosine, snippet_of(id)))
    return sort_descending_by_cosine(candidates)[:k]
```

## 5. Grade confidence (know when to abstain)

```
function classify_confidence(candidates, keyword_ids, query_tokens):
    if len(candidates) >= 2:
        top1 = candidates[0].cosine
        margin = top1 - candidates[1].cosine
        floors_pass = (top1 >= TOP1_FLOOR) and (margin >= MARGIN_FLOOR)
        for c in candidates:
            corroborated = (c.id in keyword_ids) or overlaps(query_tokens, c.snippet)
            if   not floors_pass:   c.tier = "low"      # whole neighbourhood weak
            elif corroborated:      c.tier = "high"     # two methods agree
            else:                   c.tier = "medium"
    else:                                               # single candidate, no margin
        c = candidates[0]
        if   c.cosine <= TOP1_FLOOR + MARGIN_FLOOR: c.tier = "low"
        elif corroborated(c):                       c.tier = "high"
        else:                                       c.tier = "medium"
    return candidates
```

## 6. Blend with the keyword search

```
function blended_search(query, k, keyword_bundle):
    fuzzy = fuzzy_search(query, k)
    if keyword_bundle is None:
        keyword_bundle = keyword_search(query)          # self-sufficient
    keyword_ids, keyword_scores = extract(keyword_bundle)
    keyword_scores = min_max_normalise(keyword_scores)

    confidence = classify_confidence(fuzzy_only(fuzzy), keyword_ids, tokens(query))

    rows = []
    for id in union(ids(fuzzy), keyword_ids):
        final = 0.45*cosine(id) + 0.30*keyword(id) \
              + 0.15*graph(id)  + 0.10*recency(id) + 0.00*importance(id)
        if confidence[id] == "low": final *= 0.4        # demote
        rows.append((id, final, confidence[id]))

    # low-confidence sinks below everything trustworthy
    return sort(rows, key = (tier != "low", final), descending)[:k]
```

## 7. Shadow mode (run silent until trusted)

```
function search(query):
    result = keyword_search(query)          # user sees EXACTLY this, unchanged
    render(result)

    if dowser_enabled():                    # off by default during evaluation
        try:
            fuzzy   = fuzzy_search(query, k = 20)
            blended = blended_search(query, k = 20, keyword_bundle = result)
            log({ query, keyword: result, fuzzy, blended, latencies })  # study later
        except:
            pass                            # a fuzzy failure must never break search
```

## Off switches

```
KIPPLE_FUZZY_ENABLED = 0     # global kill, checked before any import
config.enabled       = false # persistent off without an env var
drop the 3 vector tables     # full removal; records are untouched
```
