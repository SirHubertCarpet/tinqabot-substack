# The disk sentinel : pseudocode

Readable pseudocode for the hourly disk watchdog: a cheap fast pass every run, an
expensive scan when warranted, a rule table that classifies large items, and
banded alerting that stays silent until it must not.

## Configuration

```
SYSTEM_DRIVE = "/"              # tiny built-in drive, treated as full
BULK_DRIVE   = "/mnt/bulk"      # large external drive, build-shaped things live here
WATCHED      = [SYSTEM_DRIVE, BULK_DRIVE]

# fullness bands for the system drive (percent used)
BANDS = [ ("ample",    0,  74),
          ("tighten", 75,  79),
          ("tight",   80,  89),
          ("critical",90, 100) ]

ALERT_FLOOR_PCT = 1.0          # ignore wobble smaller than this
ALERT_FLOOR_MB  = 500          # and smaller than this absolute change
SCAN_STALE_HOURS = 12          # force a scan if none in this long
SCAN_JUMP_GB     = 1.0         # force a scan if fullness jumped by this much
```

## Rule table (data, not code)

```
RULES = [
  # action       matches a path like...
  ("STAY",        "/etc/**"),            # operating system, never touch
  ("STAY",        "/usr/**"),
  ("CLEAN",       "**/.cache/<pkg-mgr>"),# safe to delete, regenerates
  ("CLEAN",       "**/tmp/**"),
  ("MOVE",        "**/models/**"),       # belongs on the bulk drive
  ("MOVE",        "**/.cache/<compiler>"),
  ("MOVE",        "**/venvs/**"),
  # anything large that matches nothing above -> INVESTIGATE (handled below)
]
# STAY rules are checked FIRST so a cache that sits under an OS path is left alone.
```

## Entry point: runs hourly from cron

```
function main():
    state = load_state()           # persisted between runs; gates de-duplication

    fast = run_fast(state)         # cheap: just fullness, every run
    save observed band into state for each drive

    if should_scan(state, fast):
        run_scan(state)            # expensive: walk + classify + recommend

    save_state(state)
```

## Fast pass: the cheap hourly check

```
function run_fast(state):
    results = {}
    for drive in WATCHED:
        pct, free_mb = read_usage(drive)        # how full, how much free
        band         = band_for(pct)
        results[drive] = (pct, free_mb, band)
        maybe_alert(state, drive, pct, free_mb, band)
    return results
```

## Deciding whether to run the expensive scan

```
function should_scan(state, fast):
    if no scan yet today past the morning anchor:   return true
    if hours_since(state.last_scan) > SCAN_STALE_HOURS: return true
    for drive in WATCHED:
        if band_changed(state, drive):              return true
        if fullness_jumped(state, drive) >= SCAN_JUMP_GB: return true
    return false
```

## Scan pass: walk, classify, recommend (never act)

```
function run_scan(state):
    findings = []
    for drive in WATCHED:
        for item in largest_items(drive, top_n = 20):
            action = classify(item.path)
            findings.append({ path: item.path,
                              size: item.size,
                              action: action })       # MOVE / CLEAN / STAY / INVESTIGATE

    move_now = [ f for f in findings if f.action in ("MOVE", "CLEAN") ]
    report(move_now)            # write recommendations; a human decides
    state.last_scan = now()
    # NOTE: the sentinel emits recommendations only. It moves and deletes nothing.

function classify(path):
    for (action, pattern) in RULES:     # STAY rules first, by table order
        if path matches pattern:
            return action
    return "INVESTIGATE"                 # large and unknown -> ask a human
```

## Banded alerting: silent by default, escalating only when real

```
function maybe_alert(state, drive, pct, free_mb, band):
    if band == "ample":
        return                          # silence is the correct output

    if not crosses_floor(state, drive, pct, free_mb):
        return                          # ignore mere wobble

    if band == state.last_band[drive]:
        return                          # de-dup: already alerted at this band

    if band == "tighten":   write_inbox_note(drive, pct)      # quiet channel
    if band == "tight":     push_to_phone(drive, pct)         # louder
    if band == "critical":  speak_aloud(drive, pct)           # last resort

    state.last_band[drive] = band       # remember, so we fire once per change

function crosses_floor(state, drive, pct, free_mb):
    moved_pct = abs(pct - state.last_pct[drive])
    moved_mb  = abs(free_mb - state.last_free_mb[drive])
    return moved_pct >= ALERT_FLOOR_PCT and moved_mb >= ALERT_FLOOR_MB
```

## The whole idea in three sentences

The fast pass is cheap, so it runs every hour and catches slow leaks early.
The scan is expensive, so it runs only when something looks like it changed,
and it recommends rather than acts. Alerting stays silent while there is room
and escalates one step at a time, so that when the house finally speaks aloud,
the warning is worth heeding because every quieter option was already spent.
