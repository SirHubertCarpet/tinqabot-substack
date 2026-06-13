# Pseudocode: the glance (situational pre-flight)

Run once, synchronously, immediately before composing any reply. Cheap reads
only. Emits a compact status block the assistant reads before it writes a word.

```
function glance():
    out = []

    # 1. Handoff freshness: always report a status line
    handoff = read_text(HANDOFF_FILE)
    if handoff is empty:
        out.add("!! HANDOFF EMPTY: write session context now")
    else:
        age = now() - mtime(HANDOFF_FILE)
        if   age > 2h:  out.add("!! HANDOFF STALE (" + hours(age) + "): update now")
        elif age > 45m: out.add("HANDOFF: getting stale (" + mins(age) + ")")
        else:           out.add("HANDOFF: ok (updated " + mins(age) + " ago)")

    # 2. Side-channel notes: urgent first, and urgent STAYS until acked
    urgent = sidenotes(kind = "stop", acknowledged = false)
    if urgent:
        out.add("!! URGENT NOTE: STOP AND READ")
        out.add_each(urgent)                 # persists every run until ack
    for note in unread_sidenotes():          # ordinary notes: show once
        out.add(note)
        advance_cursor(note)

    # 3. Credential health: don't promise an action through a locked door
    for service in external_services():
        if not token_valid(service):
            out.add("!! AUTH WARNING: " + service + ": re-auth needed")

    # 4. Time-gap: a long silence means the world moved
    gap = now() - last_interaction_time()
    if gap > GAP_THRESHOLD:
        out.add("[TIME GAP] " + human(gap) +
                " since last turn: re-read the clock, re-check assumptions")

    # 5. Location: from an actual device fix, never from calendar intent
    fix = latest_device_fix()                # {lat, lng, observed_at, kind}
    if fix is None:
        out.add("LOCATION: no fix today yet")
    else:
        # 'departed' from a tracker means "left its last STOP", not
        # "set off on a planned journey". Never promote one into the other.
        out.add("LOCATION: " + describe(fix) +
                "  (as of " + fix.observed_at + ", source=" + fix.kind + ")")

    # 6. Ambient context LAST: regenerate the rolling brief if stale
    if age(BRIEF_FILE) > BRIEF_STALE_WINDOW or not exists(BRIEF_FILE):
        regenerate_brief()                   # calendar + weather + deadlines + flags
    out.add(read_text(BRIEF_FILE))

    return join(out)
```

## The discipline the glance enforces downstream

The glance only *gathers*. These three rules govern how the assistant is allowed
to *use* what it gathered. They are the reason the pre-flight is worth running.

```
# RULE A: the opening snapshot is a rumour.
# The state handed in with the question may be stale by the time you answer.
# Re-read live state at the last possible moment; answer from now.

function state_claim(subject):
    reading = read_live(subject)             # not the turn-start snapshot
    assert reading.observed_at is not None
    assert reading.source     is not None
    return reading

# RULE B: every state claim carries a timestamp AND a source.
function render(reading):
    age = now() - reading.observed_at
    if age < FRESH:
        return plain(reading.value) + " (" + reading.source + ", just now)"
    else:
        # old facts must LOOK old, and be hedged
        return hedged(reading.value) +
               " (" + reading.source + ", as of " + reading.observed_at + ")"

# RULE C: a direct human observation outranks your own probe.
function reconcile(probe_reading, user_observation):
    if user_observation contradicts probe_reading:
        # the probe is the suspect: stale cache, dead token, wrong id,
        # a sensor wedged in a state it left hours ago.
        trust(user_observation)
        recheck_via_canonical_tool(probe_reading.subject)   # never a one-off
        return user_observation
    return probe_reading
```

## Notes

- **Order is load-bearing.** Urgent notes and auth failures are surfaced before
  ambient context, so the assistant never answers cheerfully while a stop-flag or
  a dead token sits unseen.
- **Idempotent and side-effect-light.** The glance reads and reports; the one
  write it permits itself is regenerating the rolling brief when it has gone
  stale. It never changes the state it is observing.
- **`unknown` is a valid answer.** When no fresh, sourced reading supports a
  claim, the honest output is "unknown / unverified, as of <time>", never a
  confident guess dressed as a fact.
