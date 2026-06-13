# Voice Output Discipline

> How a voice assistant makes itself heard in a busy room without trampling the audio, and the rule that decides when it is allowed to speak unprompted at all.

This document describes two cooperating mechanisms in a home voice system: **room-audio ducking** (becoming audible without shouting) and the **piggyback rule** (when a non-urgent spoken alert is permitted to fire). It is a public, de-identified companion to the article *How To Speak Without Butting In*. It describes the design, not anyone's home.

## The problem

A house that speaks lives in a room that is rarely silent. Music, video, a podcast, and a loud parrot may all be playing already. Two failures are common in off-the-shelf systems:

1. **It shouts over the top.** The assistant speaks at full volume into whatever was already playing, so two sources compete and neither is intelligible.
2. **It speaks on a timer.** A non-urgent reminder fires at a moment chosen by the machine, interrupting the person cold, instead of waiting for a natural opening.

The design below addresses both. The first is a question of *how* to be audible; the second is a question of *when* it is allowed to speak at all.

## Part 1 — Ducking: becoming audible without trampling the room

Before the assistant speaks locally, it **ducks** the room: every other audio source fades down to a low floor, the assistant says its piece on top, and everything fades back up the instant it finishes.

### Smooth fade, not a hard mute

The dip is a gradual fade (about one second), not an instant mute. A hard cut is its own jolt, a stab of silence that draws attention before any information arrives. A gentle fade reads as the room politely leaning back to let the voice through.

### Adaptive floor

The depth of the dip is not fixed. A loud room needs a deeper duck than a quiet one, or the spoken line drowns. The system measures the ambient loudness of the room and picks the floor to match: louder room, deeper duck.

The critical subtlety is that **the assistant's own voice is part of the noise.** A naive loudness measurement taken while the assistant is speaking hears itself, concludes the room is deafening, and ducks everything to the floor. The fix is to measure loudness from a signal that is deliberately deaf to the assistant's own speech, so the duck responds to the room as it is *without* the assistant, not to the assistant describing the room.

### Two coordinated fade mechanisms

Not every source is directly controllable. Some audio (local music, mixer channels) can be faded directly. Other audio plays through a separate device on a different transport, which cannot be faded directly and must instead be asked to **pause itself** while the assistant talks and resume afterwards.

That means two independent mechanisms, a direct fade and an external pause, that must agree on when speech starts and ends. If they disagree, one can lift the room back up while the other is still mid-sentence. They coordinate through a shared **lease**: a small on-disk record meaning "a duck is in progress, do not undo it yet." Only the operation that started the duck may end it. Overlapping ducks join the existing lease rather than starting a competing one.

### Orphan recovery: surviving a mid-sentence death

If the process holding the duck is killed mid-sentence (crash, restart, power blink), the naive outcome is a room left pinned in silence forever, every source held down with no live owner to release it.

To prevent this, the system writes a **pre-duck snapshot** to disk before ducking: the level of every source as it was before the dip. On a clean finish, the snapshot is deleted. If a fresh process starts and finds an old snapshot still present, with no live owner, it knows a duck was orphaned and restores every level exactly as recorded. The room can be interrupted in the middle of a word and still be tidied up on the way back in.

## Part 2 — The piggyback rule: when is it allowed to speak at all

Ducking answers *how* to be audible. The harder question is *when* the assistant may speak when nobody asked it to.

### The rule

> A non-urgent spoken alert is never permitted to fire unprompted on a timer. It must **piggyback** on an interaction the person is already having: a button they pressed, a question they asked, a card they ticked.

The reminder rides in on the back of an interaction that was already happening, rather than arriving cold at a moment of the machine's choosing.

### Why it changes everything

This is a restriction on *when*, not on *what*. An alert that fires on a timer arrives at a moment chosen by the alert. An alert that waits for an interaction arrives at a moment chosen by the person, because they were already engaged and already receptive to one more small thing. The same sentence, delivered the second way, stops being an interruption and becomes an answer: "by the way, while you're here."

In practice, most non-urgent reminders make no sound at all. They appear quietly as a waiting card, and speak only when the person next engages with that stack of their own accord. A muted non-urgent announcement is **deferred to a card** rather than spoken, so it surfaces on the person's next interaction instead of out of the blue.

### The one exception: safety

There is exactly one class that does not wait. **Safety-critical** alerts (a dying smoke-alarm battery, a person at a door who should not be there) speak immediately, over anything, because piggybacking assumes the message can afford to wait for a better moment, and some messages cannot. The system distinguishes "you might like to know" from "you need to know now," and is only ever quiet about the first.

## Design principles, distilled

1. **Fade, don't mute.** A gradual dip is less startling than a hard cut and reads as courtesy, not alarm.
2. **Measure the room, not yourself.** Adaptive ducking must exclude the assistant's own voice from its loudness measurement, or it self-drives into the floor.
3. **One lease, one owner.** Multiple fade mechanisms coordinate through a shared lease so they start and end the duck together; only the starter ends it.
4. **Survive your own death.** A pre-duck snapshot on disk lets a killed process's duck be recovered, so the room is never left pinned in silence.
5. **Piggyback, don't blurt.** Non-urgent speech rides on an interaction the person initiated. Only safety overrides this.

The companion pseudocode walks through the duck-with-recovery and the piggyback gate as plain algorithms.
