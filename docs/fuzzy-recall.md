# Dowser: Semantic Recall over the Knowledge Base

Dowser is a meaning-based search layer that sits in front of the knowledge base's
existing keyword search. Where the keyword pass matches the literal words in a
record, Dowser matches what a record is *about*, so a half-remembered query can
still find a fact whose wording does not overlap with the query at all.

It is named for the dowser's forked stick: the system's job is to sense where a
relevant record lies even when nothing on the surface points to it.

## The problem it solves

Keyword search is a literalist. A query for `wildflower` returns records that
contain the substring `wildflower` (after space and case normalisation), and
nothing else. This is fast and exact, and it fails precisely when the user does
not have the original wording. A note filed as "the meadow walk in June" and a
later query of "wildflowers with a friend" share almost no characters, so the
keyword pass returns nothing.

The mismatch is structural: the store holds exact words; the user arrives with an
impression. Dowser closes that gap by comparing *meaning* instead of *spelling*.

## How meaning becomes a number

Dowser uses a sentence-embedding model. The model maps any short text to a fixed
vector of 384 floating-point numbers. The geometry of that vector space is the
whole trick: texts with similar meaning land close together, even when they share
no words. "The meadow walk in June" and "wildflowers with a friend" end up near
each other because the model was trained on a very large corpus to place related
ideas nearby.

Vectors are L2-normalised, so similarity reduces to a dot product: cosine
similarity in the range minus one to one, higher meaning closer.

## Pre-built index, on-the-spot query

Embedding text is the slow part, so it is done once, ahead of time. Every fact
and every entity in the knowledge base is embedded and stored in a vector index
that lives in the same database file as the records themselves. There is no
separate vector service to run or back up.

At query time only the user's one short query is embedded on the spot. The search
then asks the index for the stored vectors nearest the query vector. A nearest-
neighbour lookup over pre-computed vectors is fast; the cost that would dominate
(embedding thousands of records) has already been paid overnight.

A nightly job re-embeds any record whose text has changed (detected by hashing
the embed-text) and removes vectors whose source record was deleted, so the index
stays in step with the knowledge base without a full rebuild each night.

## The embed daemon

The embedding model is large and takes several seconds to load from disk. Paying
that cost on every search would add a multi-second tax to every lookup in the
system, including lookups that have nothing to do with semantic recall.

The fix is a long-lived helper process: the embed daemon. It loads the model into
memory once at startup and then listens on a local socket. Any component that
needs a text embedded sends a short request over the socket and gets the vector
back in roughly a tenth of a second, because the model is already warm. The slow
load happens once, at boot, never per query. If the daemon is unavailable, callers
fall back to loading the model in-process, so correctness never depends on the
daemon, only speed.

A subtle failure to avoid: the daemon's own embed path must call the *in-process*
embedder directly, not the shared helper that prefers the socket. Otherwise the
daemon, asked to embed, would try to call back into itself over the socket it is
busy serving, and deadlock.

## Confidence and abstaining

A meaning-search is generous: in a dense vector space, *something* is always
somewhat close, so a naive version will always return a plausible-looking
neighbour. A confidently wrong answer is worse than an honest blank.

Dowser therefore grades each result's confidence rather than trusting raw
proximity:

- **Top-1 strength.** How close is the best match in absolute terms? A weak best
  match means the whole neighbourhood is unreliable.
- **Margin.** Is the second-best suspiciously close to the best? Two near-identical
  near-misses usually indicate a vague region, not a real answer.
- **Corroboration.** Did the keyword pass independently surface the same record, or
  does the record share a query token? Agreement between two different methods is
  stronger evidence than either alone.

Low-confidence candidates are demoted and sink below the trustworthy structured
results.

## Shadow mode

While Dowser is still being evaluated it does not change what the user sees. It
runs alongside the real search, computes its own and a blended ranking, and logs
both next to the structured result for later study. User-visible output stays
byte-identical to the keyword-only system. Only after a logged evaluation shows
the new layer earns its place does it move to the front of house.

## Blending the two retrievers

The blended ranking fuses the two signals with a weighted linear combination
(meaning-similarity weighted most heavily, keyword score next, with room for graph
proximity, recency, and importance). On a judged set of real conversational
queries the two retrievers proved complementary: keyword search kept the widest
net (best recall), while the blend was clearly best at putting the single correct
answer at the very top. That top-of-list precision is the case the blend exists
for.

## Off switches

Dowser is built to be fully disengageable. A single environment flag disables it
globally before any of its machinery is even imported; a persistent config flag
does the same without an environment variable; and the entire vector layer is
three additive tables that can be dropped with no change to the underlying
records. The keyword search never imports any of the semantic machinery at the top
level, so the base system runs unchanged with the feature off.
