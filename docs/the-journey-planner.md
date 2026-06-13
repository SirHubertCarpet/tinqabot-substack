# The Journey Planner

A clean public explainer of how the home system decides when its owner has to leave for a calendar event, and how it nudges them out of the door on time. Companion to the article *Leaving On Time Without Thinking About It*.

This describes the design, not anyone's life. Names, addresses, and times are illustrative.

## The problem

A calendar tells you when an event starts. That is almost never the moment you need. By the time "10:30, the dentist" is relevant, leaving is already overdue. The actionable quantity is the *leaving time*, and nobody computes it reliably, least of all a distracted human doing mental arithmetic under time pressure.

The journey planner exists to compute that one number and then act on it.

## Four loosely coupled timelines

The wider journey system runs four independent tracks that reinforce each other but do not depend on each other:

1. **Calendar-driven**: a plan becomes a chain of calendar events, the key one being a `LEAVE` marker, which triggers spoken departure alerts.
2. **GPS-driven**: the phone streams location breadcrumbs; stop detection turns clusters of stationary points into recognised stops.
3. **Weather-driven**: outdoor events get a transparent weather overlay.
4. **Location-driven**: detected stops are matched against a knowledge base of known places.

This explainer covers the first track: planning the journey and getting the owner out of the door. You can plan a journey with no GPS, or track GPS with no plan; the tracks are deliberately separable.

## Step 1: estimate the drive, pessimistically

The planner asks a routing engine (an open-source one, not a commercial maps API) for the driving duration between origin and destination. That raw figure is an idealised, traffic-free best case and is treated as untrustworthy.

The planner inflates it with a fixed, deliberately blunt safety margin:

```
real_minutes = (engine_seconds / 60) * 1.5 + 5
```

- The **1.5 multiplier** absorbs traffic, lights, and route friction.
- The **flat +5 minutes** absorbs the non-driving overhead: parking, walking in, the fact that arriving is never instantaneous.

A clear-run estimate of 15 minutes becomes a 27.5-minute plan. The virtue is not accuracy; it is being reliably too cautious, which gets you there on time when a precise estimate would not.

**Per-leg override.** On routes the owner knows well (long uninterrupted stretches where 1.5x is genuinely too pessimistic), an experiential number can be supplied to replace the modelled estimate for that leg. Suspicion is the default; the override is the exception.

## Step 2: walk backwards from the fixed point

The planner anchors on the thing that actually matters: *be at the destination by time T*. It then walks backwards to derive the departure time.

```
leave_time = arrive_by
             - sum(real_drive_time of every leg)
             - sum(dwell at every intermediate stop)
             - prep_window
```

A trip can be a single hop or a string of stops (a pickup, an errand, the destination, a drop-off). Each leg gets its own pessimistic drive estimate; each intermediate stop gets a dwell budget. The result is laid onto the calendar as a chain, the head of which is a `LEAVE` event sitting at the exact derived minute, carrying the whole route.

The inversion is the point: the calendar now answers "when do I leave?" instead of "when does it start?". Only the former was ever actionable.

## Step 3: three graduated alerts

A leaving time sitting silently on a calendar is no use to someone who does not look at calendars. So the `LEAVE` event comes and finds its owner, three times, firming up each time. A recurring job watches for `LEAVE` events and fires:

| Lead time | Alert | Side effects |
|---|---|---|
| **20 min** | "Get ready", spoken with the drive estimate | Gather the items needed for the trip and surface them; ready the house for departure |
| **15 min** | "Nudge", names what is still unpacked | Lists the still-uncollected items |
| **5 min** | "Time to go", with the drive length | Final list of anything outstanding |

There is intentionally no fourth alert and no escalation into nagging. Past a threshold, alerts become noise, and a system that becomes noise gets ignored wholesale, losing its useful moments along with the redundant ones. Restraint is a feature.

## Step 4: suppress when already departed

The single most important behaviour has no visible output. If the owner has already left, the alerts must stop. A "time to go" announcement to an empty room is the precise failure mode that makes people abandon the whole system: a reminder that fires when it is wrong does not merely fail once, it teaches you to distrust the times it is right.

So before each alert, the planner checks a departure signal (a home/away gate, derived from live location, with a safe "assume home" default on missing data). If the owner is gone, the alert is suppressed.

```
if already_departed():
    skip_alert()
```

## Design principles worth stealing

- **Model the human, not the road.** The pessimism is calibrated to the person who is always five minutes slower than they think, not to the traffic.
- **Derive the leave time; never hand-compute it.** Anchor on the fixed point and walk backwards.
- **Restraint over completeness.** Three alerts, then silence. More would be ignored.
- **The best alert is often the one not sent.** Suppress when wrong, or the whole system loses trust.
