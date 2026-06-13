# Pseudocode — the dispatch gate

Companion to article **#016 — The Gate That Says: Not Yet**.

The load-bearing idea: before acting on a non-trivial task, route it to a domain, gather the people and tools and known failure modes, run the domain's preflight checks, and return a verdict. A separate, narrow enforcement hook can then hard-refuse the small set of actions that are genuinely hard to undo. Domain names and tools here are illustrative.

## The routing table

A fixed, additive map. Each domain knows its trigger keywords, its tools, the knowledge-base queries worth running, the docs to read, the preflight checks, and the scar tissue.

```
DomainSpec:
    keywords         # words in the task that route to this domain
    tools            # [(name, one-line description)] — existing tools for this job
    kb_queries       # searches to pre-run for this kind of task
    docs             # architecture notes to surface
    preflight_steps  # [(label, command, severity)]  severity = warn | block
    failure_notes    # hard-won "ways this has gone wrong before"

ROUTING_TABLE = {
    "journey":        DomainSpec(...),
    "email":          DomainSpec(...),
    "kb_write":       DomainSpec(...),
    "kb_read":        DomainSpec(...),
    "calendar":       DomainSpec(...),
    "home_automation":DomainSpec(...),
    "build_install":  DomainSpec(...),
    "background_code":DomainSpec(...),
    ...   # additive: new domains appended as the system grows
}
```

## The briefing call (the manual, deep half)

Run before a non-trivial task. Read-only. Gathers, never changes.

```
function butler(task_text):
    glance()                                  # current time, weather, calendar, alerts

    domains = match_domains(task_text)        # keyword routing; a task may hit several
    people  = resolve_people(task_text)       # deterministic alias match, NO model call
    places  = resolve_places(task_text)

    briefing = new Briefing()
    verdict  = PASS

    for person in people:
        if person.is_ambiguous():             # name -> multiple entities
            briefing.flag_ambiguity(person)   # surfacing is load-bearing, not optional
            verdict = max(verdict, WARN)
        else:
            briefing.add(person.name, person.pronouns, person.address)

    for d in domains:
        spec = ROUTING_TABLE[d]
        briefing.add_tools(spec.tools)            # so we don't hand-build what exists
        briefing.add_failure_notes(spec.failure_notes)   # read the scars first
        for (label, cmd, severity) in spec.preflight_steps:
            rc = run_readonly(cmd)
            if rc != 0:
                briefing.add_warning(label)
                verdict = max(verdict, WARN if severity == "warn" else BLOCK)

    print(briefing)
    return verdict        # 0 PASS | 1 WARN (proceed, address warnings) | 2 BLOCK (stop)
```

Resolution is deliberately deterministic:

```
function resolve_people(text):
    out = []
    for token in candidate_names(text):
        matches = knowledge_base.alias_lookup(token)   # exact/alias, not fuzzy-LLM
        out.append(Person(token, matches))             # matches may be 0, 1, or many
    return out
```

## The lightweight auto-briefing (always runs)

The deep call above is a tool, and a tool you must *remember* to call cannot cure forgetting to look things up. So a lightweight version runs automatically before every turn: resolver plus domain routing only, no glance, no network. It injects its briefing and stays silent when nothing resolves.

```
on every_user_turn(text):
    domains = match_domains(text)
    people  = resolve_people(text)
    if not domains and not people:
        return SILENT                 # zero-noise on the common case
    inject_context(brief_of(domains, people))   # advice into the assistant's context
    # NOTE: this half only INJECTS. It cannot stop a tool call. See the gate below.
```

## The enforcement gate (the doorman)

Advice that can be walked past is decoration. A separate pre-action hook intercepts commands *before they run* and can hard-refuse. It is narrow by design: it blocks only the consequential, hard-to-undo class and routes it to a deliberate approval path.

```
on before_shell_action(command):
    for segment in split_into_shell_segments(command):
        if not segment_runs_privileged(segment):     # word-boundary + position parse
            continue
        if is_allowed_safe_path(segment):            # the one pre-approved routine op
            continue
        if goes_through_approval_tool(segment):      # already on the deliberate path
            continue
        DENY(segment)                                # exit 2: hard refuse
        suggest_approval_path()                      # route to the PIN/confirm step
        return

    ALLOW    # everything ordinary and read-only passes without a murmur
```

```
# Detection catches wrapped/quoted/absolute forms of the privileged verb,
# but treats the bare word-as-data as harmless:
segment_runs_privileged("deploy the unit")      -> false
segment_runs_privileged("bash -lc 'PRIV ...'")  -> true
segment_runs_privileged("echo PRIV")            -> false   # the word as data, not a call

# Out of scope BY DESIGN: deliberate self-obfuscation (indirection, eval, base64).
# A static hook cannot trace that. This is a guard against accidental haste,
# not an adversarial sandbox.
```

## The discipline it encodes: propose, then wait

The whole machine exists to enforce one order of operations against enthusiasm.

```
function handle_reported_problem(report):
    findings = investigate_readonly(report)   # logs, searches, snapshots — no approval needed
    proposal = describe(findings, proposed_change)
    present(proposal)
    decision = await_explicit_go_ahead()      # a problem report is NOT a green light
    if decision != "yes":
        return                                # stop here; the change is not authorised
    apply(proposed_change)                    # only now do we cross from looking to changing
```

## The whole point

Notice what the gate never does: it does not act faster, and it does not think better. It refuses to let action begin before the people are resolved, the right tools and old scars are on the table, the preflight checks have run, and (for the moves that cannot be taken back) a human has said go. The cleverness is entirely in *when* it permits the next step. The step itself is narrow, well-fenced, and the fence is the feature.

---

*See also: [`../docs/the-gate-that-says-not-yet.md`](../docs/the-gate-that-says-not-yet.md), [`../docs/00-overview.md`](../docs/00-overview.md), [`../ARTICLE_INDEX.md`](../ARTICLE_INDEX.md).*
