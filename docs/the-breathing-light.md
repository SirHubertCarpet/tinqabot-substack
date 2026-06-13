# The Breathing Light

A public companion to the Substack article *Status You Can See From Across The Room*.

The breathing light is an ambient status indicator: a single smart bulb that pulses
gently when there is an unactioned item waiting, and rests at normal brightness when
there is not. No sound, no screen, no badge. Just a slow visual heartbeat at the edge of
the room. It is the lowest-bandwidth notification surface in the house, reserved for soft
reminders that matter but can wait.

## Design goals

1. **Non-interrupting.** The signal must be noticeable on a glance but must never force a
   context switch. A sound demands attention now; a breathing light waits to be seen.
2. **Glanceable.** Standing across the room, you should be able to tell at once whether
   something is waiting, without reading anything.
3. **Legible at any brightness.** The pulse has to stay visible whether the room is lit
   bright or dim.
4. **Deferential.** If a human adjusts the bulb manually, the system yields immediately and
   adopts that new brightness as its resting state.
5. **Quiet by default.** When nothing is waiting, the controller does as close to nothing as
   possible.

## The pulse

When there is an unactioned item, one bulb breathes:

- **Dim down** over a one-second transition, to a target below the resting brightness.
- **Hold** dim for one second.
- **Restore** to the exact resting brightness over one second.
- That is one cycle. Three cycles run back to back, then a few seconds of stillness, then
  three more. Three breaths, a pause, repeat.

The light only ever dims and restores. It never pushes brightness above whatever the resting
value was, so the rest state is always exactly what the user had the bulb set to. The breath
is a departure from normal and a return to it, which is what reads as breathing rather than
flickering or strobing. Transitions are always eased over time; an instant brightness jump is
visually jarring and is never used.

### Proportional depth

The depth of each breath scales with the resting brightness. At a bright baseline the bulb
dims by a large amount; at a dim baseline it dims by only a little. This keeps the breath
visible across the full range. A fixed-size dip would either be invisible in a dark room or
overpowering in a bright one. By scaling the dip to the baseline, the signal stays legible
everywhere. A hard floor stops the dim target collapsing to "off" at very low baselines.

### Wait-until-bright gate

The controller will not begin breathing until the bulb is on *and* its current brightness is
above a minimum threshold. A breath nobody can see is worse than no breath: it burns cycles
and conveys nothing. If items are waiting but the bulb is off or too dim, the controller idles
and re-checks. When the room later brightens (someone walks in and raises the lights), the
controller discards its stale idea of "resting brightness" and adopts the new, higher value as
its baseline, then begins. It breathes around where the bulb is *now*, not where it was when
it first stirred.

## Single source of truth: the bulb itself

The hardest behaviour to get right is what happens when a human adjusts the bulb mid-pulse.

A naive controller keeps an internal idea of the resting brightness and forces the bulb back
to it on every cycle. If the user turns the bulb up to read by, the controller drags it back
down on its next breath, and the two fight all evening.

The fix is to treat **the bulb's live state as authoritative** and the controller's stored
baseline as merely a witness. On each cycle the controller queries the bulb's actual current
brightness. If the bulb is sitting somewhere the controller did not put it (beyond a small
hysteresis margin that absorbs transition overshoot), the controller concludes a human
intervened, surrenders, and adopts the observed brightness as the new resting value. The
controller's memory can be stale; the bulb cannot lie about its own state. When they disagree,
the bulb wins.

A hysteresis margin prevents false positives: small differences caused by a transition still
in flight are ignored, so only a real human adjustment triggers a baseline change.

## Quiet when idle

When there is nothing waiting, the controller does almost nothing. It is an infinite loop with
two states, pulsing and idle:

- **Idle:** the bulb rests at its baseline. The controller sleeps, waking only to re-check
  whether something has appeared.
- **Pulsing:** items are waiting; the controller runs the three-breath cycle.

To avoid a busy poll, state changes are surfaced through a small append-only event file. The
controller does not read its contents; it only watches the file's modification time. When the
list of waiting items changes, that file is touched, the modification time moves, and the
controller wakes within a fraction of a second instead of waiting out a full poll interval.
This gives near-instant response to changes while keeping the idle loop almost free.

## Why one bulb

Pulsing two bulbs in unison looked obvious and turned out worse: mesh-network latency between
devices made the two breaths drift out of step, producing a jerky, uneven animation. A single
bulb breathes smoothly. The signal is "something is waiting," which one bulb conveys
perfectly; a second adds nothing but desynchronisation.

## The colour question

The natural extension is colour: a calm hue for a gentle reminder, an alarming one for
something pressing, so the urgency itself is glanceable. It is deliberately not built. The
current light says exactly one thing, "something is waiting," in one tone, and does it
honestly. Adding colours and rhythms risks turning a restful ambient signal into another
dashboard that demands interpretation, which defeats the purpose. The light earns its place by
being almost nothing. Restraint is the feature.

## Summary

The breathing light is a study in saying as little as possible. One bulb, one signal, eased
transitions, depth that scales to the room, a refusal to start until it can be seen, and an
absolute deference to the human who reaches over and turns the dial. Most of the engineering
went not into the breathing but into deciding how little the light was allowed to say.
