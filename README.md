# Kipple Index

A personal cognitive prosthetic.

I have Inattentive ADHD. If only I'd been more attentive. Oh wait.

This repository is the open notebook for **Kipple Index** — a personal AI scaffold I've been building and using since early 2026. It is *not* a clone-and-run framework. There is no installer, no Helm chart, no Discord community. It is one person's working second brain, exposed to daylight so other people building similar things can borrow what worked, mock what didn't, and skip the cul-de-sacs.

## What's actually in this repository

This is the **public mirror**. The private working tree contains things that have no business being on the internet — a SQLite knowledge base of personal facts, a decade of journal entries, financial records, the unredacted carousel rules. The public mirror carries the **architecture, the philosophy, and the pseudo‑code** — the parts of the system that survive being read by strangers.

- [`ARTICLE_INDEX.md`](./ARTICLE_INDEX.md) — the planned series of articles about the project. ~500 entries across 14 sections, each one anchored to a real component, document, or incident. Articles cross‑link to the matching architecture doc and pseudo‑code file below.
- [`docs/`](./docs/) — architecture docs for the subsystems that have stabilised enough to write down. New docs land as their corresponding articles land. Start with [`00-overview.md`](./docs/00-overview.md).
- [`pseudocode/`](./pseudocode/) — paragraph‑by‑paragraph walkthroughs of the load‑bearing tools, written to be readable rather than runnable. Marked `[PCODE]` in the article index. Empty until the first walkthrough article lands.

## What's deliberately not here

- The SQLite database itself. The schema is documented; the contents are not.
- Personal journals, Logbooks, photos, financial records.
- Operational credentials, IPs, hostnames where it matters, anything resembling a secret.
- The full live tree of helper scripts. Most of them are too entangled with one person's house to be useful to anyone else.

If you find something in this repo that should not be here, tell me at the email listed on the Substack and I will fix it within the hour.

## Why this exists

Every time I read a "how I built my second brain" post, I learn nothing about the actual architecture — only what the author wanted the system to make them look like. The articles in [`ARTICLE_INDEX.md`](./ARTICLE_INDEX.md) try to do the opposite: each piece is grounded in a real, named component, with the pseudo‑code in this repository and the bug history in the published commit log. The goal is for the next person building one of these to be able to read the post, click through to the architecture doc, click through to the pseudo‑code, and have all three agree with each other.

If you are building a personal AI of your own and find any of this useful, I would consider that an excellent use of my Sundays.

## Licence

Documentation and prose: [CC BY‑SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) — attribute Sir Hubert Carpet, share alike.

Code and pseudo‑code: [MIT](./LICENSE) — do whatever you like, don't sue me.

## Status

| Item | State |
|---|---|
| Article series | First post landing imminently. See [Tinqabot on Substack](https://tinqabot.substack.com/). |
| Architecture docs | Trickle release, one per landing article. |
| Pseudo‑code | Same: lands with the article that explains it. |
| Public mirror cadence | Mirrored from the private working tree when meaningful changes land. |

The point of this repository is not to be complete. It is to be *true* — every word in it is grounded in code that actually runs.
