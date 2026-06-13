# Pseudocode: The Journey Planner

The core algorithm behind *Leaving On Time Without Thinking About It*: derive a departure time for a calendar event, lay it on the calendar, and nudge the owner out of the door without nagging.

Written for clarity, not as runnable code. Constants and signals are illustrative.

```
CONSTANTS:
    DRIVE_MULTIPLIER   = 1.5      # inflate the routing engine's optimistic estimate
    DRIVE_FLOOR_MIN    = 5        # flat overhead: parking, walking in, arriving isn't instant
    DEFAULT_TRAVEL_MIN = 10       # fallback if routing fails entirely
    PREP_MIN           = 15       # getting-out-of-the-door window
    DEFAULT_DWELL_MIN  = 20       # time spent at an intermediate stop
    ALERT_OFFSETS      = [20, 15, 5]   # minutes-before-leave to fire each alert
```

## 1. Pessimistic drive estimate

```
function real_drive_minutes(origin, destination, leg_override=None):
    if leg_override is not None:
        return leg_override                  # owner knows this road; trust experience

    seconds = routing_engine.duration(origin, destination)
    if seconds is None:                      # routing failed
        return DEFAULT_TRAVEL_MIN

    raw_minutes = seconds / 60
    return raw_minutes * DRIVE_MULTIPLIER + DRIVE_FLOOR_MIN
```

## 2. Derive the leave time by walking backwards

```
function plan_journey(stops, arrive_by):
    # stops is the ordered list of legs, ending at the destination.
    # arrive_by is the immovable anchor: "be there by T".

    total = 0
    origin = HOME
    for stop in stops:
        leg = real_drive_minutes(origin, stop.location, stop.real_minutes_override)
        total += leg
        if stop is not the final destination:
            total += stop.dwell or DEFAULT_DWELL_MIN
        origin = stop.location

    leave_time = arrive_by - total - PREP_MIN
    return leave_time
```

## 3. Lay the calendar chain

```
function publish_chain(stops, leave_time):
    events = []
    events.append(make_event("PREPARE", at=leave_time - PREP_MIN, dur=PREP_MIN))
    events.append(make_event("LEAVE",   at=leave_time, route=full_route(stops)))
    for stop in stops:
        events.append(make_event(stop.name, at=stop.arrival, location=stop.address))
    events.append(make_event("ARRIVE home", at=return_time))
    calendar.create_all(events)             # idempotent; re-publish replaces the tagged chain
```

The `LEAVE` event is the trigger the alert engine watches for. Everything else in the chain exists for navigation and logging.

## 4. The graduated alert engine (runs on a short timer)

```
function on_tick(now):
    for event in calendar.events_today():
        if not is_leave_event(event):       # match on the LEAVE marker
            continue

        if already_departed():              # the crucial suppression, see below
            continue

        minutes_left = (event.start - now) in minutes

        for offset in ALERT_OFFSETS:        # 20, 15, 5
            if minutes_left has just crossed offset AND not already_fired(event, offset):
                fire_alert(event, offset)
                mark_fired(event, offset)   # dedup: each alert fires once per event
```

```
function fire_alert(event, offset):
    drive_min = estimate_drive(event.destination)

    if offset == 20:
        speak("Get ready. About " + drive_min + " minutes to drive.")
        items = gather_trip_items(event)    # what to take
        surface(items)
        ready_house_for_departure()         # assume departure is imminent

    else if offset == 15:
        speak("15 minutes. " + count_unpacked(event) + " items still to grab.")
        list_unpacked(event)

    else if offset == 5:
        speak("Time to go. " + drive_min + " minute drive.")
        list_remaining(event)
```

## 5. Suppress when already departed (the important part)

```
function already_departed():
    # A reminder that fires when it is wrong teaches the owner to distrust
    # the times it is right. So if they have already left, say nothing.

    location = live_location_or_none()
    if location is None:
        return False                        # safe default: assume still home, alert
    return distance(location, HOME) > HOME_RADIUS
```

## Why it is shaped this way

- **Anchor on the fixed point, derive everything else.** The destination time is immovable; the leave time falls out of it. Never hand-compute the leave time.
- **Pessimism is deliberate.** A precise estimate of a distracted person's arrival is precisely wrong. The blunt 1.5x + 5 models the person, not the road.
- **Three alerts, then silence.** A fourth becomes noise, and noise gets the whole system muted.
- **The best alert is the one not sent.** Suppression on departure is what keeps the system trusted rather than ripped out.
