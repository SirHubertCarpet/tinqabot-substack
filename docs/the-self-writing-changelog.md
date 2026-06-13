# The self-writing changelog : architecture

The companion architecture note for article **#010 — A House That Writes Its Own Changelog**. Pseudocode walkthrough: [`../pseudocode/the-self-writing-changelog.md`](../pseudocode/the-self-writing-changelog.md).

## What it is

Three cooperating layers that let the system answer one question reliably: *what changed, and when?*

1. **A commit discipline** — every meaningful change to the running code lands as a version-control commit that must arrive bundled with its own documentation and a written log entry. No silent edits.
2. **An integrity watchdog** — a scheduled scan over every repository the system owns, checking for the specific ways a repo quietly rots, and announcing health even when nothing is wrong.
3. **A unified event log** — a single append-only stream that almost every component writes to as it acts, so the whole system's activity can be reconstructed as one ordered timeline.

The first records *deliberate change*. The second guards the *integrity* of that record. The third captures *everything that happened*, so cause and effect can be recovered after the fact.

## Layer 1 — the three-in-one commit discipline

A bare commit message is too easy to make worthless ("fixed stuff"). So a pre-commit gate enforces a rule: any commit that touches the running code must also stage

- an update to the relevant **architecture document** (the prose that explains the part being changed), and
- a new entry in the **logbook** (a dated, plain-language record of what was done and learned).

Code, but no architecture doc staged → the commit is **blocked**. Code, but no logbook entry → **blocked**. The gate inspects the set of staged paths and refuses anything that fails the rule. An emergency override exists for genuine fire-fighting, but using it prints a visible marker, so even *skipping* the discipline is itself recorded.

The discipline exists because human judgement ("I'll document it later") is unreliable. Taking the option away in advance is the whole design.

## Layer 2 — the integrity watchdog (the "sentinel")

It was born from a near-miss: a repository holding days of real work had silently lost its link to its safe remote copy. The work existed in exactly one place, with no backup, and nothing had warned anyone for weeks.

The watchdog runs on a schedule and walks every tracked repository, raising named findings rather than failing silent. The catalogue of checks includes:

| Check | Detects |
|---|---|
| `no_remote` | Repo has commits but no remote to push to (single point of failure) |
| `unpushed` | Local work not yet copied to the remote; escalates if it has sat too long |
| `behind` | A newer version exists elsewhere and the local clone has fallen behind |
| `stale_dirty` | Tracked changes left uncommitted past a staleness threshold |
| `secret_risk` | Files shaped like secrets (keys, tokens, private keys) that should never be committed |
| `fetch_failed` | The integrity check itself could not reach the remote (auth or network) |

It also writes a small **rebuild manifest** on every scan: for each repo, its remote URL, branch, and current revision. If a drive dies, that manifest is the recipe for re-cloning every project to its last-known state. The manifest lives inside a tracked repo, so it is itself backed up by the discipline it serves.

### Silence is not reassurance

The design principle is that a clean state must be *announced*, not merely implied by the absence of an alarm. The system's morning roll-call says "storage good, backups good, git good" out loud. An absence of bad news and an active confirmation of good news look identical right up to the moment they diverge catastrophically. The watchdog never passes in silence.

## Layer 3 — the unified event log

Code changes are the formal record; the unified log is the *behavioural* record. Almost every component drops a small, structured line into one shared stream as it acts: a reminder fired, a door opened, the overnight journal assembled, a button pressed.

Two properties make it safe:

- **Additive** — logging calls sit alongside whatever the component already did; they never replace existing behaviour.
- **Fail-soft** — if the logger errors, the component carries on. Logging can never break the thing it is logging; the worst case is a gap, never a crash.

Because everything funnels into one timeline, the system can be asked a question *across all of it at once*: not "what does this one component think happened," but "what happened, everywhere, in this window, and in what order." When something breaks, the day is reconstructed from this single stream rather than reassembled from a dozen components that each kept private, disagreeing notes.

## Why it works

Each layer covers a different failure mode and they reinforce each other:

- The **discipline** catches you skipping the explanation.
- The **watchdog** catches you skipping the discipline (or the repo rotting underneath it).
- The **log** catches everything else that merely *happened*.

The intelligence is not in any clever algorithm. It is in deciding, in advance, exactly what is allowed to change quietly (nothing) and building small, dull machinery that enforces it without negotiation.

---

*See also: [`00-overview.md`](./00-overview.md), [`../pseudocode/the-self-writing-changelog.md`](../pseudocode/the-self-writing-changelog.md), [`../ARTICLE_INDEX.md`](../ARTICLE_INDEX.md).*
