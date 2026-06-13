# Pseudocode — Voice Output Discipline

Two algorithms: the **duck-with-recovery** that makes the assistant audible without trampling the room, and the **piggyback gate** that decides whether a non-urgent alert is allowed to speak at all. De-identified; describes the design, not anyone's home.

---

## 1. Speaking with a duck (the happy path)

```
function speak_in_room(text):
    duck = begin_duck(reason = "speech")     # fade room down, may be shared
    try:
        synthesize_and_play(text)            # blocking: returns when audio ends
    finally:
        end_duck(duck)                        # fade room back up (owner only)
```

The duck wraps the speech so the fade-up cannot happen before the words finish. `finally`
guarantees the room is restored even if playback raises.

---

## 2. begin_duck — fade down, adaptively, and coordinate

```
LEASE_FILE        = "duck_lease"          # shared coordination record
PREDUCK_SNAPSHOT  = "preduck_levels"      # for crash recovery
FADE_SECONDS      = 1.0

function begin_duck(reason):
    existing = read_lease(LEASE_FILE)
    if existing and not expired(existing):
        # Another duck is already active. Join it rather than competing.
        # Do NOT re-snapshot: levels are already ducked, not original.
        extend_lease(existing, reason)
        return { joined: true, token: existing.token }

    # We are the first/owner duck.
    token = new_token()                       # e.g. process_id + timestamp
    write_lease(LEASE_FILE, token, until = now() + lease_window, reason)

    # Snapshot every source's level BEFORE touching it, for recovery.
    levels = read_all_source_levels()
    write_snapshot(PREDUCK_SNAPSHOT, { levels, token, ts: now(), owner_pid: my_pid() })

    floor = adaptive_floor()                  # see below

    # Two independent fade mechanisms, started together:
    fade_directly_controllable_sources(target = floor, seconds = FADE_SECONDS)  # async
    ask_external_device_to_pause()            # device we cannot fade directly

    return { joined: false, token: token }
```

### Adaptive floor — measure the room, not yourself

```
function adaptive_floor():
    # CRITICAL: use a loudness signal that excludes the assistant's OWN voice,
    # or the measurement self-drives to the floor every time it speaks.
    rms = read_ambient_loudness_excluding_own_speech()
    if rms is missing or stale:
        return DEFAULT_FLOOR

    # Louder room -> deeper duck. Piecewise-linear between two anchor points.
    if rms <= QUIET_RMS:  return SHALLOW_FLOOR      # quiet room: gentle dip
    if rms >= LOUD_RMS:   return DEEP_FLOOR         # loud room: deep dip
    return interpolate(rms, QUIET_RMS, LOUD_RMS, SHALLOW_FLOOR, DEEP_FLOOR)
```

---

## 3. end_duck — only the owner restores

```
function end_duck(duck):
    if duck.joined:
        return                                # joiners never restore; the owner does

    lease = read_lease(LEASE_FILE)
    if lease is null or lease.token != duck.token:
        return                                # a newer operation took over; leave it alone

    wait_for_down_fade_to_settle()            # avoid racing the async fade-down
    fade_directly_controllable_sources(target = restore, seconds = FADE_SECONDS)
    ask_external_device_to_resume()

    delete(PREDUCK_SNAPSHOT)                   # clean exit: no orphan to recover
    delete(LEASE_FILE)
```

---

## 4. Orphan recovery — runs at process startup

```
# Called once when the speech process starts, BEFORE accepting new work.
function recover_orphaned_duck():
    snap = read_snapshot(PREDUCK_SNAPSHOT)
    if snap is null:
        return                                # nothing to recover

    if process_alive(snap.owner_pid):
        return                                # owner still running; it will resume
    if age(snap.ts) < GRACE_SECONDS:
        return                                # too fresh; could be an active duck mid-setup

    # A previous duck was killed mid-sentence. Put the room back as recorded.
    restore_all_source_levels(snap.levels)
    ask_external_device_to_resume()
    delete(PREDUCK_SNAPSHOT)                   # delete even on failure, to avoid a recovery loop
```

The room can therefore be interrupted in the middle of a word and still be tidied up
on the next start. No live owner is required to release the duck.

---

## 5. The piggyback gate — when may an unprompted alert speak

```
function deliver_alert(alert):
    if alert.is_safety_critical:
        # Smoke, intrusion, etc. Cannot wait for a better moment.
        speak_now(alert)                      # ducks + speaks immediately, over anything
        return

    if alert.was_requested_by_person:
        # The person pressed/asked/ticked. This IS an interaction; ride it.
        speak_now(alert)
        return

    # Non-urgent and unprompted: NEVER blurt on a timer.
    # Defer to a quiet waiting card. It will speak only when the person next
    # engages with the card stack of their own accord ("by the way, while you're here").
    defer_to_card(alert)
```

### The interaction that later carries it

```
# Triggered when the person actually engages (button press, query, tick).
on_person_interaction(interaction):
    handle(interaction)                       # do the thing they asked for first
    pending = cards_waiting_to_be_spoken()
    if pending is not empty:
        speak_now(pending)                    # the deferred alert rides out on this interaction
```

---

## Why these two pieces belong together

`speak_now` always ducks first (section 1), so *whenever* a line is spoken it is spoken
cleanly over a lowered room. The piggyback gate (section 5) governs *whether and when*
a non-urgent line gets to reach `speak_now` at all. Ducking is the manners of being
audible; piggybacking is the manners of choosing the moment. The machine controls the
first. The person controls the second, except when the building is on fire.
