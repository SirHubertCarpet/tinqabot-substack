# The session handoff

> How a stateless assistant carries work across sessions it cannot remember.

## The problem

Each assistant session is stateless. A restart, a dropped connection, or a closed terminal ends the current mind entirely; the next one starts with no knowledge of what came before. There is no growing long-term memory inside the session. The only durable thing is what the previous session deliberately writes down.

So continuity is not a memory feature. It is a message-passing protocol between one session and the next.

## The note

Continuity rests on a single handoff file. The departing session overwrites it in place; the arriving session reads it before doing anything else. The file is not free-form. It has a fixed set of named compartments, and a small writer utility refuses to save it unless every compartment is present (an empty compartment must still be filled with "none"). Forcing the shape is what makes the note useful rather than a mood piece.

## The three states that matter

The hard-won structure is the distinction between three states of work:

- **Fixed** is done, shipped, and locked in. It carries an explicit instruction: do not re-diagnose this. A symptom that reappears later is treated as a new fault, not as evidence the original fix never happened. Without this, every fresh session re-investigates settled history, because it cannot tell "fixed" from "claimed fixed by a stranger."
- **Open** is genuinely still to do. The real backlog.
- **In the air** is the dangerous middle: actions already taken and irreversible, but not yet confirmed (a service restarted, a message sent, a state file rewritten). Each entry is stamped with the exact time it happened.

The middle state exists because of a specific failure. A session performed an irreversible action, handed over, and the next session, reading only a two-state (fixed or open) note, saw the action sitting in limbo and prepared to perform it again. Filing "done but unconfirmed" under "open" makes the next session repeat irreversible work. The separate compartment, with timestamps, stops that.

## Situational awareness, not a task list

The note's most counter-intuitive rule is about tone, not content. An early version instructed the next session to "pick up where we left off." That is correct for a sequential restart and wrong for a concurrent peer.

Sessions can overlap. A second session may boot while the first is still alive and actively working. If the handoff reads as an order, the second session starts doing the first session's live job, duplicating it, or sweeps the first session's half-finished changes into something permanent.

The fix reframes the handoff as situational awareness, not a task queue. The arriving session is told: here is what is going on, someone may still be doing it, read it and wait to be told what is yours. The information is unchanged; only the imperative framing is removed. A new session is also expected to check recent project history before touching any file the note mentions, in case it was already shipped by another session.

## The standing snapshot

The handoff covers the moment of handover. A separate mechanism, a rolling context snapshot, covers the present. It regenerates every few minutes from the clock, calendar, weather, task lists, and system health, and the assistant consults it before every response, not only at startup.

The two must not be confused. The handoff is memory of the past; the snapshot is a window onto now. A system that reads a stale snapshot as live will make confident, wrong claims about the present moment, such as reporting a journey already begun when it has not.

## Why it works

The protocol does not extend the assistant's memory; nothing can. It makes each act of forgetting survivable. The next mind needs only the note, correctly shaped: what is done (so it is not redone), what is open (so it is not missed), what is in the air (so it is not fired twice), and the instruction to read the whole thing as a guest rather than an heir.
