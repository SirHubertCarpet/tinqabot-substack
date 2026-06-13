# Pseudocode - heliograph delivery confirmation

The core idea in code form: carry a notification from phone to speaker, and at every
hop refuse to call it delivered until that is actually true. Names and platform
details are abstracted; this is the shape of the logic, not a runnable program.

---

## Phone side - the listener and the shipper

```
# LISTENER: fires on every incoming phone notification.
on_notification(n):
    record = wrap(
        fields = [n.app, n.time, n.title, n.body, n.subtext, n.channel],
        begin  = MARKER_BEGIN,      # unusual token, e.g. an unlikely bracketed string
        sep    = MARKER_SEP,        # so commas/newlines in the body cannot split fields
        end    = MARKER_END,
    )
    append_line(BUFFER_FILE, record)

    # NOTE: do NOT enable the OS "suppress multiples" option on this listener.
    # A messaging app re-posts a whole conversation in a burst on each new message;
    # suppression keeps only the FIRST child, which may be a stale re-post, and
    # silently drops the genuinely new message. The house de-dupes downstream, so
    # suppression here only ever loses real messages.


# SHIPPER: fires when BUFFER_FILE changes (file-watch trigger).
on_buffer_changed():
    wait(SETTLE_MS)                 # let a same-instant write finish landing

    contents = read(BUFFER_FILE)

    # CONFIRMATION GUARD 1: never ship-and-clear an empty buffer.
    # Without this, a clear() looked like a change, re-triggered the shipper,
    # and posted an EMPTY body that the server accepted with a 200 - a success
    # report for a message that did not exist, which then erased the evidence.
    if is_blank(contents):
        log("skip: buffer empty, nothing to ship")
        return

    response = http_post(HOUSE_ENDPOINT, body=contents)

    if response.ok:
        # CONFIRMATION GUARD 2: only clear what we actually read and sent.
        # Re-read and remove exactly `contents`; if the listener appended more
        # while we were posting, that tail survives for the next cycle.
        remove_prefix(BUFFER_FILE, contents)
    else:
        log("post failed; leaving buffer intact for retry", response.status)
```

---

## House side - the server accepts and fingerprints

```
on_post(request):
    raw = request.body

    # CONFIRMATION GUARD 3: fingerprint EVERY accept, including the empty ones.
    # An arriving void must leave a trace, or a recurring empty-delivery bug
    # vanishes into an undifferentiated count of "successful" deliveries.
    audit_append(SERVER_AUDIT, { ts: now(), bytes: len(raw), body: raw })

    if not contains(raw, MARKER_BEGIN):
        # legacy / malformed / empty - recorded above, but produces no log row
        return ok()                 # ack so the phone does not spin on retries

    for block in find_all(raw, MARKER_BEGIN .. MARKER_END):
        fields = split(block, MARKER_SEP)
        if any(field contains MARKER_SEP):       # split failed - should be impossible
            sentinel_dump(block)                 # log loudly, keep the raw block
            continue
        append_line(today_log(), to_row(fields))

    return ok()
```

---

## Watcher - decide, speak only safely, and keep durable proof

```
tail(today_log()) as row:

    if not relevant(row):           # housekeeping noise, self-sends, muted threads
        continue

    if debounced(row.sender, within = DEBOUNCE_WINDOW):
        continue                    # apps fire 2-3 notifications per real message

    urgent = is_flagged_urgent(row)

    if asleep() and not urgent:     # asleep() reads the LIGHTS, not the clock
        file_to_morning_tray(row)
        route = "held"
    else:
        hint = topic_hint(row.body) # 3-4 words, NO names/dates/verbatim content
        speak(sender = row.sender, hint = hint)   # body is NEVER read aloud
        route = "spoken"

    # CONFIRMATION GUARD 4: durable record of what we ACTUALLY acted on,
    # written OUTSIDE the directory the nightly prune trims. The proof of a
    # delivery must outlive the delivery.
    acted_log_append({ ts: now(), sender: row.sender, route: route })
```

---

## Upstream health sentinel - so a stall is not silent

```
every 30s:
    write_atomic(HEARTBEAT, { ts: now(), tailing: is_tailing(), last_row_at: last_row_ts })

status_tile():
    if not service_active():                      return "down"
    if age(HEARTBEAT) > HEARTBEAT_STALE:          return "hung?"
    if age(newest_capture_file()) > CAPTURE_STALE: return "capture pipe down?"
    return "healthy"
    # The most reliable link is the one nobody watches, right up until it stalls.
    # CAPTURE_STALE is calibrated above the worst NORMAL quiet gap, not guessed.
```

---

## The one rule underneath all four guards

```
# A delivery is not proven by the sender's confidence that it sent,
# nor by the receiver's 200 OK. It is proven only when:
#   (a) what was sent was non-empty and was the real message, and
#   (b) a durable record of the act survives long enough to be checked.
# Treat every "delivered" as a claim until both hold.
```
