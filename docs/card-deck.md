# The card deck

The companion architecture note for article **#002 — A Deck of Cards With Nothing to Look At**. Pseudocode walkthrough: [`../pseudocode/card_deck.md`](../pseudocode/card_deck.md).

## What it is

The card deck is the system's primary output surface, and it has no screen. It is a small, ordered set of *cards* — single context-dependent prompts — exposed through one physical button and a speaker. Press the button; the house speaks the front card. That is the whole interface. There is no list to scroll and no dashboard to feel guilty about.

The deck is not a todo list. A todo list is an archive of everything you have ever meant to do. The deck is a picture of *now*: only the cards that are relevant at this moment are in it, and most of the day it is nearly empty.

## The card template

Every card is stamped from one template with a fixed shape, so the machinery that handles cards reads the *shape*, never the meaning:

- `label` — the line that gets spoken
- `group` — which part of the day it belongs to (e.g. morning, evening)
- `trigger` — the gate that lets it appear (e.g. a particular light turning off)
- `every_n_days` — recurrence
- `forgotten_after` — hours after which an untouched card counts as missed
- `depends_on` / `delay_minutes` — optional: another card that must be done first, and how long to wait after it

Because the shape is uniform, adding a new kind of card is a data change, not a code change. This is the "Top Trumps" model: identical cards, different values printed on them.

## The lifecycle

A card earns its place by a trigger and loses it by an eject rule. The deck is rebuilt whenever the house changes — a light flips, a card is ticked, a timer fires — and the rebuild is idempotent: the same inputs always yield the same deck.

Triggers read the **house, not the clock**. A light going off in the evening is a more honest signal that the day has changed shape than the time on a clock, which has no idea whether the day is actually over. Each card also carries its own eject rule, so once its moment passes it removes itself rather than lingering as a reproach.

Some cards `depend_on` another and only surface once it is done, sometimes after a fixed delay. The deck holds the dependency, watches for the prerequisite to be completed, waits out the delay, and only then promotes the dependent card.

## Two contracts that earn their keep

**One source of truth.** Exactly one place answers "has this card been done today, and when". Every other component reads from it and is forbidden a second opinion. The first version of this system spread that answer across several files, each of which believed it was in charge; a card could be simultaneously done, not done, and overdue depending on which file you asked. Collapsing to a single ledger is what made the deck reliable.

**Eject and recreate, never hide.** A card is live in the deck or it does not exist. There is no third "present-but-hidden" state. Hidden cards become ghosts: invisible, holding stale references, occasionally speaking from a moment that has passed. To remove a card you eject it; to bring it back you build a new one. Enforcing this with no exceptions made a whole class of bug impossible rather than merely rare.

## The button

The button has two gestures. A single press speaks the current front card (highest priority, or "nothing right now"). A double press ticks the front card done, which updates the single ledger once and triggers a rebuild — at which point the card ejects itself and any dependents it was blocking may appear.

## Why it works

The intelligence lives in the small uniform template and one honest ledger. The loop that reads them is short and deliberately dull, and dull is exactly what you want from the thing that decides whether you remembered the start of your day.

---

*See also: [`00-overview.md`](./00-overview.md), [`../pseudocode/card_deck.md`](../pseudocode/card_deck.md), [`../ARTICLE_INDEX.md`](../ARTICLE_INDEX.md).*
