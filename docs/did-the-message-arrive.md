# Heliograph - proving a notification actually arrived

The heliograph is the part of the house that carries phone notifications to a
speaker, so the resident hears the messages that matter without watching a screen.
This document describes its shape and, more importantly, the delivery-confirmation
discipline that grew up around it after several silent losses.

The guiding principle: **a delivery system is not allowed to assume a message
arrived. It must be able to prove it, and it must treat its own confident success
report as suspect until the proof is in.**

---

## The pipeline

```
phone notification (text / message / camera-person event)
    |
    v
phone-side listener  - catches the notification, wraps each one in begin/end
    |                  markers with an unusual field separator so commas and
    |                  newlines inside a message body cannot tear the fields apart
    v
phone-side shipper   - notices the buffer file changed, posts it to the house,
    |                  then clears the buffer for the next message
    v
house server         - unwraps the bundle, appends one line to the day's log
    |
    v
watcher (tail + react) - decides per message: speak, hold, or file silently
```

### What the watcher speaks

The body of a message is **never read aloud**. Banks send security codes by text;
people send things not meant for a room with guests in it. The house announces only
the sender and a short topic hint (three or four words) produced by a small language
model, with no names, dates, or verbatim content. Enough to decide whether to pick up
the phone; never enough to embarrass anyone.

### Quiet hours

Whether to hold a message back is decided from the **state of the house**, not the
clock. If the main lights are off, the house assumes the resident is asleep and routes
non-urgent messages silently to a morning tray. A message flagged urgent breaks
through. Reading "am I asleep" from the lights rather than a clock time matters when
the resident keeps irregular hours.

---

## Why silent failure is the real enemy

Every link in the chain can fail without making a sound, and a silent failure in a
delivery system is uniquely poisonous: **the absence of an alert is indistinguishable
from the absence of a message.** "Nobody messaged me" and "somebody messaged me and
the pipe ate it" both present as a quiet house. The whole confirmation effort exists
to tell those two states apart.

Three real failure modes drove the design.

### Failure 1 - the message died before it entered the system

A setting on the phone's notification listener, "suppress multiples," threw away all
but the first of a burst of near-simultaneous notifications. The messaging app
re-posts the whole conversation group in a flurry a few milliseconds apart when a new
message lands; suppression kept the first child of the flurry, which was sometimes a
stale re-post of an older conversation, and let the genuinely new message die on the
phone.

The cruelty of this fault is that **every component downstream is healthy and
innocent**. The server, the watcher, and the speaker are all fine. The message died
one step before any of them, so there is nothing in the system's own logs to find.

Fix: turn the suppression off at the source. The house already de-duplicates with a
per-sender debounce, so suppression on the phone was protecting against a problem the
house had already solved, at the cost of dropping real messages.

### Failure 2 - the success-that-lies race

The handoff from listener to shipper went through a shared buffer file: the listener
**writes**, the shipper **reads, sends, and wipes**. The write and the wipe could land
in the same instant. The shipper, triggered by the listener's write, would sometimes
read a fraction too early or clear a fraction too late, wiping the message before it
was sent.

Worse, clearing the file looked like another change, which re-triggered the shipper,
which posted the now-empty buffer. The house accepted a perfectly valid delivery
containing nothing and returned a cheerful acknowledgement, and **that acknowledgement
erased the evidence**. The logs showed a steady run of successful deliveries, all
empty, each a tombstone for a message wiped a millisecond too soon.

This is the incident that reframed the whole project. The bug was not that messages
were lost; carrying anything across a gap risks loss. The bug was that the system was
**certain it had succeeded**. A green light that lies is worse than a red light that
screams, because the rest of your trust is built on top of it.

---

## The delivery-confirmation discipline

The fixes are unglamorous on purpose.

1. **Not-empty guard before "delivered."** The shipper may not send an empty buffer
   and call it a delivery. A "the buffer is actually not empty" check stands between
   the send and the success report, so a wiped message cannot masquerade as a real one.
   The shipper also waits a beat after the change and only clears what it actually read.

2. **Raw-body capture for suspicious accepts.** The server records the raw body of
   anything it accepts, *including* the empty nothings, with a timestamp. An arriving
   void now leaves a fingerprint instead of vanishing into a success count, so a
   recurring empty-delivery pattern is visible after the fact.

3. **A record that outlives the pipe.** Every watch-and-react point keeps its own
   running log of what it actually acted on, kept outside the directory the nightly
   prune job tidies up. The evidence of a delivery survives longer than the delivery
   itself.

4. **A health sentinel on the upstream.** The most reliable trigger in the chain had
   no health monitoring, so a phone-side stall was invisible. A staleness check now
   watches both a periodic heartbeat from the watcher and the freshness of the capture
   stream, and degrades the status tile when either goes quiet for too long.

### The structural endgame

The deepest fix is to take the phone out of the critical path entirely for
security-grade events, letting the camera talk to the house directly rather than
relaying through a flurry of notifications, a shared buffer file, and a setting somebody
ticked long ago. A relay of mirrors is a fine way to hear that a friend said hello. It
is a poor thing to put in front of a smoke alarm.

---

## Design lessons

- A delivery system's own "delivered" is a claim, not a proof. Make it earn belief.
- Silent loss is worse than loud loss; the absence of an alert must be made to mean
  something you can check.
- A success acknowledgement that erases the evidence of what it acknowledged is a
  trap. Capture the raw input before you trust the verdict.
- The most reliable upstream link is the one nobody monitors, right up until it stalls.
