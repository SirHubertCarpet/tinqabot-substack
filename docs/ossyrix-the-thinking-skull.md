# Ossyrix, The Thinking Skull : Architecture

Ossyrix is an animatronic "thinking skull": a desk-mounted resin skull whose jaw, head tilt, head turn, and eyes are driven by four servo motors, choreographed into theatrical scenes. It is a sibling project to the Kipple Index, but where the Index keeps records, Ossyrix performs. This document explains the controller's design and, in particular, the one principle the whole thing is built around: the face must never be made to wait.

## Hardware shape

| Part | Role |
|---|---|
| ESP32-C3 microcontroller | The single brain. Runs one cooperative loop, no real-time OS. |
| PCA9685 PWM controller | Generates the precise pulse widths the four servos need. |
| TCA9548A I2C multiplexer | Lets several I2C devices that share addresses coexist on one bus. |
| Servos (x4) | Jaw, head tilt, head turn, eyes. |
| Current sensor on the servo rail | Measures live current draw for overcurrent protection. |
| Thermal camera | Detects the warmest blob in the room so the skull can follow people. |
| OLED display | Local status: boot phase, mode, faults. |

Two battery packs power it: a small cell in the skull for the logic and sensors, and a larger multi-cell pack in the base, behind a battery-management board, for the servos.

## The governing idea: one brain, no blocking

The ESP32-C3 has a single thread of attention. Anything that "waits" (for a network, for a slow sensor, for a timer) freezes everything else, including the servos. A frozen servo loop is a dead-looking face: a jaw stuck open, a half-finished turn. So the entire controller is written to be **non-blocking**. Nothing is allowed to sit and wait. Long operations are modelled as state machines that the main loop nudges forward one small step per pass, then moves on. The animation update runs on every pass regardless, so the face is always live.

## Animation engine

Movement is built from **keyframes** and **scenes**:

- A **keyframe** is a target pose (four servo angles), a transition time to reach it, a hold time to stay there, an easing curve, and an optional "personality" of small idle motions during the hold.
- A **scene** is an ordered list of keyframes that tells a little story.
- **Easing** (sine, cubic, and similar curves) makes movements accelerate and settle organically instead of snapping, which is the difference between "alive" and "robot".

Each loop pass the engine works out how far through the current keyframe's transition it is, applies the easing curve to that progress, and writes the interpolated angle to each servo. To avoid micro-jitter, tiny pulse changes below a threshold are suppressed (hysteresis), so the controller is not constantly nudging a servo by a fraction of a degree.

## Non-blocking wifi: look before you leap

Wifi is the classic blocking trap, and the most rewarding part to get right. Naively, you ask the radio to connect and it blocks until success or timeout. On a busy mesh network that can be many seconds of frozen face.

The design instead:

1. **Cache of known networks**, in preference order.
2. **Scan first.** Before attempting anything, scan what is actually being broadcast right now. Pick the first *cached* network that is genuinely visible. Never burn a full timeout courting a network that is not on the air.
3. **Aim at the strongest radio.** For the chosen network, target the specific access point with the strongest signal. If that access point reports "not found", release the pin so the next attempt can roam.
4. **Fallbacks last.** If no cached network is visible, rotate through a couple of hardcoded fallbacks.
5. **Generous per-attempt window.** Each attempt may take up to roughly forty-five seconds on a slow mesh, because the wait is non-blocking and costs the face nothing.
6. **Watchdog.** A supervisor checks connection health periodically. If the link has been dead too long, it tears the radio down and brings it back up, with a cooldown so it cannot thrash.

The whole connection effort runs in the gaps between servo writes. The skull keeps performing while part of its mind hunts for the router.

## Exhibition mode and the arbitration problem

**Exhibition mode** makes the skull perform scenes autonomously, optionally auto-starting at boot. Separately, **thermal tracking** lets the skull turn to follow the warmest person in the room. Both want to command the same head servos, and naively running both produces a skull that twitches, trying to obey two masters.

The resolution is a strict priority, not a negotiation:

- A **scheduled exhibition scene always wins.** While such a scene is playing, tracking suspends and keeps its hands off the servos.
- Tracking is demoted to a **content provider for idle moments.** It supplies something to look at in the gaps between scenes, rather than competing for control during them.

The implementation distinguishes an exhibition scene from an idle (tracking-driven) scene via a flag, and tracking simply returns early whenever a non-idle scene is active. One owner at a time; the twitching stops.

## Current sensing and the safety interlock

Servos are strong and the skull is heavy. A jammed movement (a snagged wire, a bound joint) lets a servo strain indefinitely and overheat. Ossyrix watches its own current draw on the servo rail in real time. If current spikes past what a healthy movement should ever require, the controller reads it as a fault and throws an **interlock**: it disables servo output and refuses to move until a human clears it.

This is a pain reflex. The skull cannot see that its jaw is stuck, but it can feel that it is straining far beyond the job, and it stops rather than burning out. A single "clear interlocks" action (a button in the web interface, or a command) resets all overcurrent and fault state at once.

## Safe boot

To avoid a violent lurch at power-on, startup is three phases: disable servo output, set all channels to their centre positions while output is still disabled, then enable output so the servos glide gently to centre from a standing start rather than snapping there mid-motion.

## Web interface

A small embedded web server (reachable over the local network) provides live servo dials, manual sliders, mode toggles, fault display, current and signal gauges, and the "clear interlocks" control. During an exhibition performance, the manual sliders disable themselves so a stray drag cannot fight the show.

## Why it is built this way

Every hard part of Ossyrix reduces to the same discipline: the single brain must never block, and when two impulses want the same servo, one must clearly yield to the other. The illusion of a thinking skull is entirely a matter of the face never freezing and never fighting itself.
