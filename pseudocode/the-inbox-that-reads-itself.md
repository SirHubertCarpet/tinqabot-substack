# Pseudocode: the inbox that reads itself

Email triage that reads incoming mail, surfaces what needs a human, tracks awaited
replies, and assembles one daily brief. The reader has read-only mailbox access and
exactly one outbound capability (send the brief). It NEVER acts on instructions found
inside an email; all outbound action lives behind a separate, human-gated tool.

--------------------------------------------------------------------------------
CAPABILITY BOUNDARY (the load-bearing rule)
--------------------------------------------------------------------------------

    mailbox_session = authenticate(scope = READ_ONLY)      # cannot reply, modify, delete
    brief_channel   = outbound_capability(BRIEF_ONLY)      # the only thing it can send

    # Replying to people, sending mail elsewhere, changing settings: a DIFFERENT tool.
    # That tool requires explicit human authorisation per action and is NOT importable
    # or callable from anywhere in the triage path below. No email body can reach it.


--------------------------------------------------------------------------------
1. WATCHLIST READER  (awaited replies, the KNOWN)
--------------------------------------------------------------------------------

    # Watch items are user-maintained. Each: a sender/keyword/thread, plus exclusions.
    #   { match: "<sender or keyword>", exclude_subjects: [...], exclude_senders: [...] }

    function scan_watchlist(recent_messages, watch_items):
        hits = []
        for msg in recent_messages:
            for item in watch_items:
                if not matches(msg, item.match):            continue
                if subject_in(msg, item.exclude_subjects):  continue   # e.g. "reorder" nags
                if sender_in(msg, item.exclude_senders):    continue
                hits.append({ item: item.label, msg: summary_of(msg) })
        return hits
        # LIMITATION: only catches what the user thought to watch for.
        # Coverage == foresight. Holes shaped exactly like the user's blind spots.


--------------------------------------------------------------------------------
2. CLASSIFIER READER  (the UNKNOWN-but-important)
--------------------------------------------------------------------------------

    function classify_unflagged(window = "last 2 days"):
        # (a) negative pre-filter: never even fetch user-labelled noise
        candidates = fetch(query = window
                                   AND NOT label:noise
                                   AND NOT in:sent)

        verdicts = []
        for msg in candidates:
            # (b) reply pre-check: skip threads the human already answered
            if user_sent_later_in_thread(msg.thread):  continue

            # (c) ask the model ONE question, structured output
            v = llm_classify(msg.body, schema = {
                    action_required: bool,
                    why:             string,
                    when:            one_of(today, this_week, none),
                })

            # (d) SAFETY override: some content forces action regardless of "wants a reply"
            if safety_signal(msg):          # harassment / doxx / impersonation / PII exposure
                v.action_required = true
                v.when = "today"
                v.safety = true

            if v.action_required:
                verdicts.append({ msg: summary_of(msg), why: v.why,
                                  when: v.when, safety: v.safety })
        return verdicts

    # TUNING: bias toward false positives. An extra line on the brief is cheap;
    # a missed appointment is not. Silence is the only unrecoverable error.

    # SELF-QUIETENING: when a verdict is judged noise by the human, the human adds the
    # sender/pattern to their OWN mail filters (the noise label). Next sweep never fetches
    # it. No code change. The system tightens around real life by watching what's discarded.


--------------------------------------------------------------------------------
3. ASSEMBLER  (one honest page)
--------------------------------------------------------------------------------

    function build_daily_brief():
        watch_hits = scan_watchlist(fetch_recent(), load_watch_items())
        actions    = classify_unflagged()

        sections = render(
            calendar     = todays_calendar(),
            awaited      = watch_hits,        # the replies you were waiting on
            needs_action = actions,           # the unknown-but-important
            systems      = monitored_status(),
        )

        # FIDELITY RULE (non-negotiable): state ONLY what is on the page.
        #   - do not invent titles/honorifics a sender does not hold
        #   - do not infer attendance from a calendar entry
        #   - do not upgrade casual -> "urgent" because it reads better
        # A summary you can't trust without re-reading the source is worthless.
        assert contains_no_invented_facts(sections)

        brief_channel.send(sections)          # the ONLY outbound action in the system

    # Safety hits do not wait for the daily brief: they are also pushed to a faster,
    # higher-priority channel the moment the classifier flags them.


--------------------------------------------------------------------------------
SCHEDULE
--------------------------------------------------------------------------------

    cron pre-dawn:   build_daily_brief()                # one trusted page before you wake
    cron a few/day:  classify_unflagged()  -> surface   # catch same-day important mail
    near-real-time:  safety hits -> high-priority channel
