# Pseudocode — the card deck lifecycle

Companion to article **#002 — A Deck of Cards With Nothing to Look At**.

This is the load-bearing loop: given the state of the house and what has already been done today, decide which cards are on the deck *right now*. Card labels here are illustrative, not real.

## The card template

Every card is stamped from a template with the same fixed shape. The deck machinery reads the shape, never the meaning — it does not know what "the bins" are.

```
Card:
    label             # what gets spoken, e.g. "the bins go out tonight"
    group             # part of the day it belongs to: "morning", "evening"
    trigger           # the gate that lets it appear, e.g. gate:dining_light_off
    every_n_days      # recurrence; 1 = daily
    forgotten_after   # hours after which an untouched card counts as missed
    depends_on        # (optional) another card that must be done first
    delay_minutes     # (optional) wait this long after depends_on is done
```

## One source of truth

There is exactly one place that answers "has this card been done today, and when". Call it the ledger. Nothing else keeps its own copy; every decision below reads the ledger, and no other component is allowed a second opinion.

```
ledger.done(card)       -> timestamp or None    # when it was ticked today
ledger.last_done(card)  -> date or None          # for every_n_days
```

## Rebuilding the deck

Runs whenever the house changes (a light flips, a card is ticked, a timer fires). It is idempotent: same inputs, same deck.

```
function rebuild_deck(house, ledger, now):
    wanted = []                                   # cards that SHOULD be live now
    for card in all_card_templates:

        # 1. Recurrence — is it even due today?
        if card.last_done is today: continue
        if not due_by_every_n_days(card, ledger, now): continue

        # 2. Trigger gate — has the house opened the door for it?
        if not gate_open(card.trigger, house): continue

        # 3. Already handled today? then it has no business on the deck.
        if ledger.done(card) is not None: continue

        # 4. Dependency + delay
        if card.depends_on:
            dep_done = ledger.done(card.depends_on)
            if dep_done is None: continue                      # prerequisite not met
            if now < dep_done + card.delay_minutes: continue   # too soon

        wanted.append(card)

    # 5. Reconcile the live deck against `wanted` — EJECT AND RECREATE.
    #    No hidden state: a card is live, or it is gone.
    for live in deck.current():
        if live not in wanted:
            deck.eject(live)            # remove it; never "hide" it
    for card in wanted:
        if card not in deck.current():
            deck.create(card)           # a fresh card, never an un-hide

    deck.order_by_priority()            # top-priority takes the front spot
```

## Why eject-and-recreate

The one rule with no exceptions: a card is live in the deck or it does not exist. There is no third "present-but-hidden" state. Hidden cards become ghosts — invisible, still holding stale references, occasionally speaking up from a moment that has already passed. To remove a card you eject it; to bring it back you build a new one. Enforcing this made a whole class of bug impossible rather than merely rare.

## What the button does

The wall button is the only input. It does not open a screen; it asks the deck for its current front card and speaks it aloud.

```
function on_button_press(deck, ledger, now):
    card = deck.front()                 # highest-priority live card, or none
    if card is None:
        speak("nothing right now")
    else:
        speak(card.label)

function on_button_double_press(deck, ledger, now):
    card = deck.front()
    if card is not None:
        ledger.mark_done(card, now)     # the single source of truth, updated once
        rebuild_deck(house, ledger, now)  # the card ejects itself; dependents may now appear
```

## The whole point

Notice what is *not* here: no per-card special cases, no second store of "done", no visibility flag. The intelligence lives in the small uniform template and one honest ledger. The loop that reads them is short and dull, and dull is exactly what you want from the thing that decides whether you remembered your morning.

---

*See also: [`../docs/00-overview.md`](../docs/00-overview.md), [`../ARTICLE_INDEX.md`](../ARTICLE_INDEX.md).*
