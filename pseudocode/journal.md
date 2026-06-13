# Pseudocode — the journal pipeline

Companion to article **#003 — The Journal That Writes Itself While I Sleep**.

This is the load-bearing idea: collect cheap fragments all day, then assemble a strictly-grounded narrative overnight. The writer is never allowed to know more than the brief tells it. Tags and examples here are illustrative.

## The fragment

A fragment is one line, written the moment something happens. It is a note to tonight's writer, not finished prose.

```
Fragment:
    ts        # timestamp, ISO
    tag       # build | life | health | mood | trip | visitor | memory
    text      # one short sentence (optional for mood-only)
    mood      # optional energy/mood label

function add_fragment(text, tag, mood, date):
    day  = date or today()
    line = { ts: now(), tag: tag }
    if text: line.text = text
    if mood: line.mood = mood
    append line to fragments_file(day)
```

## Overnight assembly

Runs on a schedule (2:45am, for *yesterday*). It builds a brief, then writes the narrative from the brief and nothing else.

```
function assemble_journal(date):
    fragments = load_fragments(date)

    bik_cal, other_cal = get_calendar_events(date)   # split: own vs shared
    weather            = get_weather(date)
    alerts             = get_home_alerts(date)

    # Self-healing: don't trust fragments to be complete.
    kb_records = knowledge_base.records_dated(date)   # what the graph logged
    uncovered  = [r for r in kb_records
                    if not mentioned_in(r, fragments)]  # gaps the notes missed

    brief = build_brief(date, fragments, bik_cal, other_cal,
                        weather, alerts, kb_records, uncovered)
    save_brief(date, brief)

    # The narrative writer may read ONLY the brief.
    narrative = write_narrative(brief)               # rule: not in brief => didn't happen
    save_local(date, narrative)
    append_to_rolling_doc(date, narrative)
```

## Calendar separation

The owner's own calendar is treated as their activities. Every other visible calendar is context only, and never narrated as something they did unless a fragment confirms it.

```
function get_calendar_events(date):
    own, other = [], []
    for cal in all_visible_calendars():
        is_owner = cal.is_the_owners()
        for ev in cal.events_on(date):
            entry = format(ev.time, ev.summary, ev.location)
            if is_owner:
                own.append(entry)
            else:
                other.append(label(cal.name, entry))   # context, not an event
    return own, other
```

In the narrative writer:

```
# Shared-calendar events are background unless independently confirmed.
for ev in other_cal:
    if not any_fragment_confirms(ev): do_not_narrate(ev)
```

## The brief is the leash

The brief is the only thing the writer reads. Everything the narrative may say must trace to a line in it.

```
function build_brief(date, fragments, own_cal, other_cal,
                    weather, alerts, kb_records, uncovered):
    section "Weather"            <- weather
    section "Own calendar"       <- own_cal          # the owner's activities
    section "Other calendars"    <- other_cal        # CONTEXT ONLY, do not narrate
    section "Fragments"          <- fragments sorted by ts
    section "Home alerts"        <- alerts
    section "Known records"      <- kb_records        # enrichment, NOT today's events
    section "Uncovered records"  <- uncovered         # fragments missed these — fold in
    section "Date warnings"      <- consistency_check(recent_records)
    return joined(sections)
```

## The date consistency check

A small guard run before writing, catching the specific error of a weekday that contradicts its date.

```
function consistency_check(records):
    warnings = []
    for r in records:
        if r mentions a weekday and a date:
            if weekday_of(r.date) != r.named_weekday:
                warnings.append("mismatch: " + r.text)   # surfaced in the brief
    return warnings
```

Authoritative-date rule, applied when deciding which day a record belongs to:

```
# The filing date wins over the wall clock.
day_for(record) = record.logged_date          # not when the keys were pressed
day_for(life_event) = life_event.ts.date()    # life events use their own time
```

## Recovery

A read-side watchdog lets a missed night be found and backfilled deterministically, instead of discovered by accident.

```
function audit(date):
    return classify(date)   # written | appended | needs_write | needs_append | needs_brief

function backfill(date):
    assemble_journal(date)  # same path, run by hand for a past day
```

## The whole point

Notice what the writer never gets to do: invent, embellish, or promote a shared-calendar event into a thing that happened. The cleverness is entirely upstream, in the brief and the cross-check. The writer itself is a narrow, well-fenced step, and the fence is the feature.

---

*See also: [`../docs/journal.md`](../docs/journal.md), [`../docs/00-overview.md`](../docs/00-overview.md), [`../ARTICLE_INDEX.md`](../ARTICLE_INDEX.md).*
