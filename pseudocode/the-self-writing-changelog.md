# The self-writing changelog : pseudocode

Companion to article **#010 — A House That Writes Its Own Changelog**.

Three small, dull pieces of machinery: a gate that refuses undocumented commits, a watchdog
that scans every repository for the ways they quietly rot, and a fail-soft event log that
turns scattered activity into one ordered timeline.

## Part 1 — the commit gate (the three-in-one rule)

Runs automatically before every commit. It only inspects the *set of staged paths*; it
does not need to understand the code. If a commit changes running code, it must also carry
its own documentation and its own logbook entry, or it is refused.

```
function pre_commit_gate(staged_paths, env):

    # Emergency hatch: explicit, and it leaves a mark.
    if env["SKIP_CHECKS"] == "1":
        print("WARNING: discipline skipped by override")   # visible in the record
        return ALLOW

    touches_code     = any(p matches "*.py" or dependency-manifest for p in staged_paths)
    has_arch_doc     = any(p matches "docs/*-architecture.md" for p in staged_paths)
    has_logbook      = any(p matches "Logbooks/LOGBOOK-*.md"  for p in staged_paths)

    if touches_code and not has_arch_doc:
        block("code changed but no architecture doc staged")    # exit non-zero -> commit aborts

    if touches_code and not has_logbook:
        block("code changed but no logbook entry staged")

    return ALLOW
```

The rule is deliberately blunt. It does not check that the docs are *good*, only that they
exist and were touched in the same commit. The point is to make "I'll write it up later"
impossible, not to grade the prose.

## Part 2 — the integrity watchdog

Walks every tracked repository on a schedule and emits *named findings*. Severity drives
whether a finding merely shows up in a status roll-call or escalates to an alert. Nothing
passes in silence: a clean repo still reports "good".

```
SECRET_SHAPES = [ "*.pem", "*token*", "*.env", "id_rsa", "credentials.*", ... ]

function scan_repo(repo):
    findings = []

    if repo.has_commits and repo.remote is None:
        findings.add(high, "no_remote: work exists in only one place")

    ahead = repo.commits_ahead_of_remote()
    if ahead > 0:
        sev = high if oldest_unpushed(repo) > STALE_HOURS else low
        findings.add(sev, "unpushed: {ahead} commits not copied to safety")

    if repo.commits_behind_remote() > 0:
        findings.add(med, "behind: a newer version exists elsewhere")

    for f in repo.modified_tracked_files():
        if age(f) > DIRTY_HOURS:
            findings.add(med, "stale_dirty: {f} uncommitted too long")

    for f in repo.untracked_files():
        if matches_any(f, SECRET_SHAPES) and not gitignored(repo, f):
            findings.add(high, "secret_risk: {f} looks like a credential")

    if not repo.fetch_succeeded():
        findings.add(med, "fetch_failed: could not reach remote")

    return findings


function watchdog_run(repos):
    report = []
    for repo in repos:
        if repo.has_file(".no-sentinel"):       # explicit opt-out for scratch repos
            continue
        report.add(repo.name, scan_repo(repo))

    write_rebuild_manifest(repos)               # see below
    announce(report)                            # "git: good" even when clean
    return report
```

### The rebuild manifest

Cheap to write on every scan, invaluable if a drive dies. For each repo it records the
recipe to restore it. The manifest lives *inside* a tracked repo, so it is itself protected
by the commit discipline.

```
function write_rebuild_manifest(repos):
    manifest = []
    for repo in repos:
        manifest.add({
            name:     repo.name,
            path:     repo.path,
            remote:   repo.remote_url,      # git clone <remote> <path>
            branch:   repo.branch,
            head:     repo.head_revision,   # git checkout <head>
        })
    save(manifest)   # restore = clone each remote, check out each head
```

### Announcing clean

```
function announce(report):
    issues = [f for repo in report for f in repo.findings]
    if issues is empty:
        say("git: good")                 # silence is not reassurance; say it anyway
    else:
        say("git: {len(issues)} issues")
        for f in high_severity(issues):
            alert(f.message)             # deduplicated so a standing issue does not re-spam
```

## Part 3 — the unified event log

Every component that does anything meaningful drops one structured line into a single shared
stream. Two rules keep it from ever causing harm: it is **additive** (added alongside
existing behaviour, never replacing it) and **fail-soft** (a logging error is swallowed, so
the component carries on).

```
function emit(service, event_type, **payload):
    try:
        line = {
            time:    now(),
            service: service,
            event:   event_type,
            data:    payload,
        }
        append_to_stream(line)           # one shared, append-only timeline
    except any error:
        pass                             # logging must never break the caller
```

Usage is a single extra line wherever something happens:

```
emit("reminders", "fired", id=card.id)
emit("door",      "opened", which="front")
emit("journal",   "assembled", date=today)
```

### Asking the timeline a question

Because everything lands in one stream, "what happened, everywhere, in this window" is a
single ordered read, not a reconciliation across a dozen disagreeing components.

```
function reconstruct(window):
    events = read_stream(where time in window)
    return sort(events, by=time)         # cause-and-effect, recovered from one source
```

## How the three fit together

```
A change to the running code
    -> commit gate: refuse it unless it carries docs + a logbook entry   (deliberate change, documented)
    -> watchdog:    on its next scan, confirm the repo is healthy and backed up   (the record stays intact)

Anything the house DOES (not just code)
    -> emit() one line into the unified stream   (so the day can be replayed in order later)
```

The discipline catches a skipped explanation. The watchdog catches a skipped discipline, or
a repo rotting underneath it. The log catches everything else that merely happened. None of
it is clever; all of it is hard to skip, which is the entire point.
