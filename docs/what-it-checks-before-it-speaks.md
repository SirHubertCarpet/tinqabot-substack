# The Glance: situational pre-flight before every response

The glance is a short, deterministic pass the assistant runs immediately before
composing any reply. Its single job is to refresh what the system believes is
true about the world, at the last possible moment, so the answer is built on the
present rather than on a snapshot that has quietly gone stale between the
question arriving and the answer being written.

It is not an intelligence layer. It does no reasoning and makes no decisions. It
gathers current state, flags anything urgent, and hands the assistant a fresh,
timestamped picture to answer from.

## Why it exists

The failure mode it defends against is the *fluent stale claim*. The picture of
the world handed to the system at the start of a turn is accurate when handed
over and frequently wrong by the time the reply is finished: a task gets
completed, a journey begins, an appointment moves, a note gets dropped, the wall
clock advances. A system that answers from the opening snapshot produces small,
plausible, confident untruths. Enough of them and the system stops being
trusted, which is the only asset it had.

The glance treats the opening snapshot as a rumour to be re-checked, not as
ground truth.

## The ordered sequence

The checks run in a fixed order, loudest-and-most-urgent first, ambient context
last. The order is deliberate: there is no value in being well-informed about
the weather while an urgent stop-note sits unread.

1. **Handoff freshness.** Is the session-continuity note present and recent, or
   stale and in need of an update? Surfaced as a status line every run.
2. **Side-channel notes.** Out-of-band notes the user can drop without
   interrupting work. *Urgent* notes stay surfaced until explicitly acknowledged
   (seen is not the same as actioned); ordinary notes appear once.
3. **Credential health.** Are the tokens that reach external services still
   valid, so the system does not promise an action through a door that has
   quietly locked?
4. **Time-gap check.** How long since the last interaction? A long silence means
   the world had time to move; a real gap trips a flag telling the assistant to
   re-read the clock and re-check its assumptions.
5. **Location.** Read from a record of where the user's device has actually been,
   with a timestamp, never inferred from a calendar's statement of intent.
6. **Ambient context (last).** Calendar, weather, deadlines, and overnight flags,
   read from a rolling brief that is regenerated if it is older than a short
   staleness window.

## Three load-bearing rules

### The snapshot is a rumour

State is re-read at the last possible moment before answering. The cost is a few
cheap reads per turn; the payoff is that the reply reflects now, not the
slightly-stale version of now that arrived with the question.

### A timestamp and a source on every state claim

No assertion about the world is allowed to travel bare. Not "the door is locked"
but "the door reported locked, two minutes ago, by the lock itself". A fact with
no age cannot be checked for staleness, and an unstaleable fact is precisely what
gets a system into trouble. The age of a fact is half the fact: fresh readings
can be stated plainly, old ones must be presented as old and hedged accordingly.

### Intent is not evidence

A calendar event that says the user is leaving is a statement of intent, not
proof they went. Position comes only from an actual device fix with a timestamp.
A calendar may suggest; a breadcrumb may confirm; the system is forbidden from
promoting a suggestion into a confirmation. This is the fix for the class of
error where the assistant narrates a journey that has not happened, and for the
subtler one where a single ambiguous field ("departed") is read as a stronger
claim than it actually carries ("left its last stop").

## Observation outranks the probe

The final principle is one the system holds against itself: if one of its own
probes contradicts something the user is directly observing, the *probe* is
presumed wrong. A script reporting on the world is a witness, and witnesses lie:
a stale cache, an expired token, the wrong identifier read off the wrong device,
a sensor wedged in a state it left hours ago. So a direct human observation is
not argued with. It is believed, and the system then re-checks through the
canonical tool for that domain (never a hand-rolled one-off) to find which of its
own readings was wrong.

## What it is not

The glance does not fuse signals into verdicts, infer unseen state, or decide
anything. That is the job of a separate, heavier layer. The glance is the cheap,
always-run pre-flight that sits in front of every reply and refuses to let the
system speak from a snapshot it has not re-checked.

The core algorithm is written out as readable pseudocode alongside this document.
