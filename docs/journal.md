# The journal

The companion architecture note for article **#003 — The Journal That Writes Itself While I Sleep**. Pseudocode walkthrough: [`../pseudocode/journal.md`](../pseudocode/journal.md).

## What it is

The journal is the system's memory of the day. Every night it produces a short prose entry narrating what happened: build work, visitors, errands, mood, journeys. The entry is written in the voice of the house's AI companion, dated, and appended to a single rolling document that accumulates one entry per day. The owner writes none of it; the assembly runs overnight, unattended.

The point is not a log of timestamps. It is readable continuity: a paragraph you can hand to "what did you do this week?" instead of a spreadsheet. For someone whose working memory does not reliably keep the thread of recent days, that paragraph is the product.

## Two phases

The journal deliberately does not reconstruct the whole day from nothing at assembly time. That would be a machine inventing a life. Instead the work splits in two.

**During the day, fragment accumulation.** As things happen, single-line fragments are appended to a per-day file. Each fragment carries a timestamp, a tag, and a short sentence. Tags are intentionally mundane: `build`, `life`, `health`, `mood`, `trip`, `visitor`, `memory`. A fragment is not finished prose; it is a note passed forward to tonight's writer.

**Overnight, assembly.** A scheduled job (2:45am) gathers the day's fragments, pulls the owner's calendar, the weather, and the home-automation alerts, and writes a structured *brief*: a dossier of everything known to have happened. Only then is the narrative written, working strictly from the brief.

## The one rule that holds it together

**If it is not in the brief, it did not happen.** The narrative writer may not add atmosphere, weather it cannot evidence, drinks nobody recorded, or moods nobody observed. It writes what the brief can prove and nothing else. An AI left free to garnish a day will produce a beautiful account of a day that did not occur; the brief is the leash.

## What it refuses to trust

**Fragments are not the only source.** The first version trusted the fragments completely, which meant trusting the owner to remember to file them, which is exactly the faculty the project exists to compensate for. A fully-logged afternoon of work once never got a fragment and silently vanished from that day's entry. The fix: assembly also queries the wider knowledge base for records dated to that day and cross-checks. Anything the knowledge base knows that the fragments missed is surfaced and folded in. The journal became self-healing, and can no longer silently drop part of the day because a note was never taken.

**Other people's calendars are context, not events.** The house can see several calendars: the owner's own, and shared ones for other people and places. Assembly separates them. An event on a shared calendar is treated as context only and is never narrated as something the owner did unless a fragment independently confirms attendance. The journal narrates one life, not the ambient schedule around it.

## The date rules

Time is where it breaks, so the rules are explicit:

- **The authoritative date is the one the record was filed under**, not the wall-clock moment of the work. A late-night session logged under Monday belongs to Monday even if the keys were still moving at 1am.
- **Life events use their own timestamp.** A visitor at 10:30 on Monday is in Monday's entry.
- **A day gets only its own records.** Sessions from another date must not bleed into this entry.
- **Every run does a consistency check** across recent records for weekday and date mismatches (a "Monday the 14th" when the 14th was a Tuesday) and flags them in the brief, so they are caught before they harden into the permanent record.

## Output and recovery

The assembled entry is saved locally and appended to a single rolling document, one dated heading per day. A read-side watchdog audits the pipeline, classifying each date as written, appended, or still needing work, so a missed night can be enumerated and backfilled deterministically rather than discovered by accident months later. Backdated recollections, something remembered after the fact, are folded into the right past day under a "later recollections" heading without rewriting the original entry.

## Why it works

The intelligence is not in the prose. It is in the strictness about what the writer is allowed to know: a brief it cannot exceed, a knowledge-base cross-check it cannot dodge, a calendar separation it cannot blur, and date rules it cannot bend. The writing, by the time all that is decided, is the easy part.

---

*See also: [`00-overview.md`](./00-overview.md), [`carousel.md`](./carousel.md), [`../pseudocode/journal.md`](../pseudocode/journal.md), [`../ARTICLE_INDEX.md`](../ARTICLE_INDEX.md).*
