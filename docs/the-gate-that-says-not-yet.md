# The gate

The companion architecture note for article **#016 — The Gate That Says: Not Yet**. Pseudocode walkthrough: [`../pseudocode/the-gate-that-says-not-yet.md`](../pseudocode/the-gate-that-says-not-yet.md).

## What it is

The gate is a mandatory dispatch step that runs before the assistant acts on any non-trivial task. The owner's image for it is plain: one door, and a butler answers it. Before doing anything consequential, the assistant states its intent; the gate hands back a briefing, runs its checks, and returns a verdict. Only then does work proceed. Trivial single-sentence answers are exempt. Everything heavier (code, writes to the knowledge base, email, calendar work, journeys, installs, background code, multi-step operations) goes through the door.

The point is not surveillance. It is the installation of a single good habit (look before you change) somewhere it cannot be skipped. The gate does not make better decisions than the assistant would; it makes the *same* decisions the assistant would make if it reliably gathered the facts first, which under load it does not.

## What the briefing contains

Given a stated task, the gate does several cheap, deterministic things:

- **Resolve the people.** Named people are matched against the knowledge base by deterministic alias matching (no model call, no GPU spend). Each resolved person comes back with their pronouns and, where relevant, address or coordinates, so the assistant never proceeds on a vaguely remembered detail.
- **Surface ambiguity.** If a name resolves to more than one person, that ambiguity is raised rather than guessed. Surfacing ambiguity is load-bearing, not optional: a confident wrong match is worse than a flagged uncertain one.
- **Recognise the domain.** The task is routed to one of a fixed set of task domains (journey, email, knowledge-base read, knowledge-base write, calendar, home automation, maintenance, build/install, background code, social publishing, legal, house geography, and so on). The routing is keyword-driven and additive: new domains are appended as the system grows.
- **Lay out the right tools.** Each domain carries a curated list of the existing tools for that kind of job, with one-line descriptions. This directly counters the most common waste: hand-building something that already exists, better made, a few folders away.
- **Read out the failure modes.** Each domain carries a hard-won list of the specific ways that kind of task has gone wrong before, captured at the moment it hurt. "Undo the first action before doing the second." "That shared calendar is not the owner's calendar." The briefing surfaces this scar tissue before the wound can be reopened.
- **Run preflight checks.** Some domains run a real read-only check before work starts (for example, searching the calendar for existing events before a journey is planned). Each check has a severity: a soft warning, or a hard block.

## From leaflet to doorman

The first version was advisory only. It produced a verdict (proceed / proceed-with-warnings / stop) but it could not actually hold the door shut. It ran as a lightweight hook that injected its briefing and always allowed the action to continue; its "binding" exit codes were honoured only on good behaviour. Advice you can walk past is interior decoration.

This is a recurring lesson. A check that depends on being *remembered* cannot cure the disease of forgetting to look things up. (A different guard in the same system once sat wired into the wrong place, dead, for over two weeks, indistinguishable from a working one.) So the gate gained teeth.

The enforcing half is a separate pre-action hook that can return a hard refusal. It is narrow on purpose. The overwhelming majority of work (looking, reading, ordinary harmless changes) passes without a murmur. The gate only plants its feet for the small set of actions that are genuinely hard to take back: privileged system changes that would alter the building rather than tidy a shelf. Those are intercepted, refused, and routed to a deliberate path with a real, intentional moment of friction (an explicit approval step). The friction is the feature. It is the half-second in which a catastrophe becomes a thought someone decided not to have.

A known, deliberate limit: a static pre-action check guards against *accidental* haste, not a determined adversary. Self-obfuscated commands (indirection, encoding, aliasing) are out of scope by design. This is a self-guard, not a sandbox.

## The discipline underneath: propose, then wait

The gate exists to enforce an order of operations that enthusiasm erodes: a reported problem is not permission to change anything. Noticing a fault, even an obvious one, even an urgent-feeling one, does not authorise rebuilding it. The required sequence is: diagnose, state in words what is wrong and what is proposed, and then **wait for an explicit go-ahead** before touching code, config, services, or stored facts. Read-only investigation (logs, searches, snapshots, replay tests) is encouraged without asking; the line being held is between *looking* and *changing*. Crossing it needs a yes, even when the fix is obvious.

The waiting feels like waste. It is not. It is the only moment in which a bad plan can still be stopped cheaply, before it has been carefully built.

## Why it works

The intelligence is not in the gate. It is in the strictness about *when* the assistant is allowed to act: not before the people are resolved, not before the domain's tools and scars are on the table, not before the preflight checks have run, and, for the consequential moves, not before a human has said go. The acting, once all that is decided, is the easy part. The hard part was admitting the door needed a keeper.

---

*See also: [`00-overview.md`](./00-overview.md), [`../pseudocode/the-gate-that-says-not-yet.md`](../pseudocode/the-gate-that-says-not-yet.md), [`../ARTICLE_INDEX.md`](../ARTICLE_INDEX.md).*
