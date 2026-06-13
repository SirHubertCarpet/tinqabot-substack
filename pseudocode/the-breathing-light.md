# The Breathing Light (pseudocode)

Plain-language pseudocode for the ambient status indicator described in the Substack article
*Status You Can See From Across The Room*. One bulb, breathing when something is waiting,
resting when nothing is. Names and values are illustrative.

## Constants

```
CYCLE_SECONDS        = 2     # one dim-and-restore breath
BREATHS_PER_GROUP    = 3     # breaths before a pause
PAUSE_SECONDS        = 3     # stillness between groups
RECHECK_SECONDS      = 30    # how often the idle loop re-checks
DEPTH_FRACTION       = 0.4   # how deep a breath dips, as a fraction of baseline
MIN_START_BRIGHTNESS = 40    # won't begin breathing below this
DIM_FLOOR            = 1     # dim target can never fall below this
BULB_MARGIN          = 10    # bulb-vs-memory difference that counts as a human adjustment
WAKE_POLL_SECONDS    = 0.25  # how often the idle loop checks the wake signal
```

## Main loop

```
function run():
    pulsing  = false
    baseline = read_resting_brightness()   # the bulb's normal level

    loop forever:
        if something_is_waiting():
            if not pulsing:
                # only begin if the breath would actually be visible
                live = read_bulb_brightness()
                if bulb_is_off() or live < MIN_START_BRIGHTNESS:
                    wait_for_change_or_timeout(RECHECK_SECONDS)
                    continue
                baseline = live            # adopt the room's current level
                pulsing  = true

            for i in 1 .. BREATHS_PER_GROUP:
                new_baseline = breathe_once(baseline)
                if new_baseline is not null:
                    # a human moved the dial mid-group; adopt and restart the group
                    baseline = new_baseline
                    break
            wait_for_change_or_timeout(PAUSE_SECONDS)

        else:
            if pulsing:
                set_brightness(baseline, eased = true)   # return to rest
                pulsing = false
            wait_for_change_or_timeout(RECHECK_SECONDS)
```

## One breath

```
function breathe_once(baseline):
    # proportional depth: deeper breath when the room is brighter
    dim_target = max(DIM_FLOOR, round(baseline * (1 - DEPTH_FRACTION)))

    set_brightness(dim_target, eased = true)    # ease down over ~1s
    sleep(1 second)                             # hold dim

    # before restoring, check whether a human has taken over the bulb
    changed = detect_human_adjustment(baseline, dim_target)
    if changed is not null:
        return changed                          # caller adopts new baseline

    set_brightness(baseline, eased = true)      # ease back to rest over ~1s
    sleep(1 second)
    return null
```

## Deference: the bulb is the single source of truth

```
function detect_human_adjustment(baseline, dim_target):
    live = read_bulb_brightness()      # ask the bulb, not our own memory
    if bulb_is_off():
        return null

    # if the bulb sits far from BOTH our resting value AND the dim target we
    # just sent, a human must have moved it. The margin absorbs the overshoot
    # of an in-flight transition so we don't fire on our own easing.
    if abs(live - baseline) > BULB_MARGIN and abs(live - dim_target) > BULB_MARGIN:
        record_resting_brightness(live, source = "human")
        return live                    # the bulb wins; adopt its value

    return null
```

## Waking quickly without busy-polling

```
function wait_for_change_or_timeout(timeout_seconds):
    # A separate part of the system appends to an event file whenever the set
    # of waiting items changes. We never read its contents: we only watch its
    # modification time. This wakes us within a fraction of a second of a real
    # change, while letting the idle loop stay almost free.
    last_mtime = mtime(event_file)
    deadline   = now() + timeout_seconds
    while now() < deadline:
        if mtime(event_file) != last_mtime:
            return    # something changed; wake immediately
        sleep(WAKE_POLL_SECONDS)
    return            # nothing changed; the timeout simply elapsed
```

## Reading whether something is waiting

```
function something_is_waiting():
    # A fast cached snapshot is kept fresh by whatever maintains the to-do set.
    snapshot = read_cached_state()
    if snapshot.age < CACHE_TTL_SECONDS:
        return snapshot.has_items       # trust the cache if recent
    return compute_waiting_items_live() # otherwise recompute from source
```

## Design notes

- **Only dims and restores.** The bulb never exceeds its resting brightness, so the rest state
  is always exactly what the user set. The breath is a dip and a recovery.
- **Eased transitions only.** Every brightness change is eased over time. An instant jump reads
  as a flicker, not a breath.
- **Proportional, with a floor.** Depth scales to baseline so the breath stays visible in both
  bright and dim rooms; the floor stops it collapsing to "off" when the room is very dim.
- **Wait until visible.** No point breathing in the dark. The gate holds until the bulb is on
  and bright enough, then adopts the new brightness as baseline.
- **The bulb wins.** Live bulb state outranks the controller's stored memory. A human turning
  the dial is detected and honoured on the next cycle.
- **One bulb.** Two bulbs drift out of sync over a mesh network and look jerky. One breathes
  smoothly and says everything that needs saying.
