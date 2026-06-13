# Pseudocode: the session handoff protocol

A stateless assistant cannot remember across sessions. Continuity is a note,
written in a fixed shape by the departing session and read like a guest's diary
by the next one. This is the core of it.

## The note has fixed compartments

```
HANDOFF_FIELDS = [
    "ACTIVE_PROJECT",      # one line: what we are working on
    "WHAT_WE_ARE_DOING",   # one or two sentences, concrete
    "FIXED",               # shipped AND confirmed this session, do NOT re-diagnose
    "JUST_DID_IN_THE_AIR", # irreversible actions taken, NOT yet confirmed, timestamped
    "OPEN",                # genuinely still to do (actions already taken do NOT go here)
    "KEY_STATE",           # decisions, paths, ids, values the next session needs
    "WHAT_TO_RESTART",     # service names, or "none"
    "NEXT_STEPS",          # priority order
    "BLOCKERS",            # waiting on a human or external, or "none"
]
```

## Writing the note (departing session)

```
function write_handoff(fields):
    # An empty compartment is still information; it must say "none".
    for name in HANDOFF_FIELDS:
        if name not in fields:
            raise Refuse("missing compartment: " + name)

    # The three states are not interchangeable.
    #   FIXED   -> done and confirmed     -> next session must NOT redo or re-diagnose
    #   OPEN    -> still to do            -> next session may pick up (if told to)
    #   IN_AIR  -> done, not yet confirmed -> next session must NOT repeat it
    for action in fields.JUST_DID_IN_THE_AIR:
        assert action.has_timestamp()   # exact time it happened

    write_atomically(HANDOFF_PATH, render(fields))   # tempfile + replace, never half-written
```

The classic bug: an irreversible action (a restart, a sent message) filed under
OPEN. The next session reads it as pending and does it twice. The fix is a
separate, timestamped IN_AIR compartment so "done but unconfirmed" is never
mistaken for "still to do".

## Reading the note (arriving session)

```
function on_session_start():
    note = read(HANDOFF_PATH)

    # CRITICAL: the note is situational awareness, NOT a task list.
    # Another session may still be alive and doing this work right now.
    present(note)                  # show the human what was going on
    acknowledge("caught up on handoff")   # state, not "resuming work"

    # Do NOT auto-start WHAT_WE_ARE_DOING, IN_AIR, or NEXT_STEPS items.
    # Wait for an explicit instruction.
    do_not = [note.WHAT_WE_ARE_DOING, note.JUST_DID_IN_THE_AIR, note.NEXT_STEPS]

    for file in files_mentioned_in(note):
        if recent_history(file).shows_already_shipped():
            skip(file)             # a concurrent session may have done it

    wait_for_instruction()
```

The phrasing matters. "Pick up where we left off" turns a peer session into an
unrequested takeover. "Here is what is going on; wait to be told what is yours"
does not.

## The standing snapshot is a separate thing

```
# Regenerated every few minutes; consulted before EVERY response, not just at start.
function context_snapshot():
    return gather(clock, calendar, weather, task_lists, system_health)

# Handoff   = memory of the past (read once, at start).
# Snapshot  = window onto now (read every turn).
# Never read a stale snapshot as live state, or you will assert a wrong "now".
```

## The whole point

The assistant never gains memory. The protocol only makes each forgetting
survivable: write the note in the right shape, separate done from open from
in-the-air, and read it as a guest rather than an heir.
