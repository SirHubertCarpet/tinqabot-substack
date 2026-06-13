# The disk sentinel : architecture

A small single-board computer runs the whole house. Its built-in system drive is tiny (on the order of fifty gigabytes). The disk sentinel is the discipline and the watchdog that keep that drive from filling, which would take everything down with it.

## The core principle: treat the system drive as full

There are two storage devices:

- **The system drive** (built into the board, small, fast). Reserved for the operating system and a handful of small files that genuinely must live there. Treated as if it is already full.
- **The bulk drive** (external, large, plugged in over a cable). Everything build-shaped lives here: machine-learning models, isolated software environments, downloaded caches, compiled artefacts, and the temporary debris of installing anything.

The rule is a one-liner: *if it is build-shaped, it goes on the bulk drive.* The difficulty is never stating the rule; it is enforcing it against tools that default to writing wherever is convenient (which is the small drive, because that is where the home folder lives).

## Enforcing the rule with signposts

Build tools that can be told where to write their data are told, through environment settings applied across the relevant shell types, to write to the bulk drive instead:

- Software-environment folders and the model store are placed on the bulk drive by default.
- Tool-specific caches (package managers, compilers, build systems) are each redirected by their own environment variable.
- The package installer is configured to refuse installation unless it is inside a properly isolated environment, which by policy can only exist on the bulk drive.

This makes the correct location the path of least resistance, so the default behaviour of a careless install lands in the right place.

## Why signposts are not enough

Signposts work until something ignores them: a tool configured before the signposts existed, writing to the old place out of habit. A stray cache appears on the small drive, then another. Because nobody watches a disk by hand, the small drive fills silently over days, and the first symptom is something important failing to write its file. A drive at eighty percent looks identical to a drive at forty percent. The failure is invisible until it is total.

## The sentinel: a two-speed watcher

A small program runs hourly. It has two modes:

| Mode | Cost | What it does |
|------|------|--------------|
| Fast pass | Cheap | Reads how full each drive is. Runs every hour because it is nearly free. Catches slow leaks early. |
| Scan pass | Expensive | Walks the drive, finds the largest items, and classifies each one against a rule table. Runs on a daily anchor, and also whenever the fast pass sees a sudden jump or staleness. |

The scan classifies each large item into one of a few actions:

- **MOVE** the item belongs on the bulk drive (for example a model that landed on the small drive).
- **CLEAN** a cache that can simply be deleted.
- **STAY** part of the operating system; leave it alone.
- **INVESTIGATE** something new and large that no rule yet covers; flag it for a human.

The sentinel never moves or deletes anything itself. It is a watchman, not a cleaner. It reports findings and recommendations; a human decides.

## Escalation bands: silence by default, speech as a last resort

A warning that always shouts trains you to ignore it. A warning that never shouts is useless. So alerting is banded by how full the drive is:

1. **Plenty of room** silence. The correct output when nothing is wrong.
2. **Tightening** a quiet note in the house's inbox, read at leisure.
3. **Tighter** a push notification to the owner's phone.
4. **Critical** the house speaks the warning aloud, the option of last resort.

Two guards keep the escalation honest:

- **Floors.** An alert must clear both a percentage threshold and an absolute amount before it counts, so a few megabytes of normal wobble never trigger anything.
- **De-duplication.** The same alarm is not raised twice for no change. State is persisted between runs so each band change fires once.

## Why it matters

The whole house runs on that one small board. Every higher-level feature sits on top of the system drive. If the drive fills, none of the cleverness can write, and all of it stops at once. Watching a small precious thing more closely than its size seems to warrant, and escalating slowly so that the eventual spoken warning is worth heeding, is the general pattern: build for the failure you cannot see coming, and get tapped on the shoulder before the floor gives way.
