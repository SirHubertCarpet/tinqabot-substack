---
platform: index
length: meta
title: Kipple Index — Article Index (draft v1)
slug: index
---

# Kipple Index — Article Index (draft v1)

A planned series of articles about the Kipple Index project, written as "we" — a human-AI collaboration. Every entry below is grounded in a real component, document, rule, incident, or service in the live system. No invented examples. Pseudocode walkthroughs are flagged `[PCODE]` and will lift directly from `tools/*.py` rather than be generated.

**Platform legend:** `sub-L` = Substack long (~800–1200w) · `sub-M` = Substack medium (~500–700w) · `fb-M` = Facebook medium (~250–400w) · `fb-S` = Facebook short (~120–200w) · `ig` = Instagram (~80–140w + tags)

**Format per row:** `NNN | platform | Title — Subtitle | summary | draws-from`

---

## A. Manifesto & Wide-Angle (1–20)

001 | sub-L | A Cognitive Prosthetic For The Permanently Distracted — What I built when I gave up trying to remember things myself | The thesis: a knowledge graph plus a small fleet of agents that read and write to it; continuity, situational awareness, low-noise output. Owns the ADHD framing directly; introduces the project from "what is this thing for" rather than from architecture. | docs/00-overview.md, docs/kb-architecture.md

002 | sub-L | Why Kipple? — The philosophy behind a personal AI scaffold | Continuity as the missing primitive of LLM tooling; memory, attribution, timestamp+source. | CLAUDE.md rule 0b, MEMORY.md rule #0

003 | sub-M | The Three-Layer Model — Source feeds repository, repository drives behaviour | Calendar/sensors/email feed an intent layer; consumers read from the layer, never the source. | feedback_source_vs_repository_of_truth.md

004 | sub-L | We Built This For Ourselves — Why personal AI works at human scale | Single-user assumptions let us optimise where multi-tenant SaaS cannot. | docs/KIPPLE_INDEX_SYSTEM_MEMO.md

005 | fb-M | A Knowledge Graph You'd Actually Talk To | Entity-attribute-relationship-fact as a vocabulary, not a database design. | docs/kb-architecture.md

006 | sub-M | What Counts As A Fact — Categories, confidence, importance, anchors | The shape of an enhanced_fact and why each field earns its keep. | docs/db_schema.md, CLAUDE.md rules 2,4,5,6

007 | fb-M | Quiet By Default — The discipline of an unobtrusive assistant | One channel one mention; alerts piggy-back on interaction; no timers. | feedback_alerts_piggyback_existing_interaction.md, feedback_no_nagging.md

008 | ig | Continuity Is The Killer Feature | One line: most AI tools forget. Ours doesn't. | MEMORY.md rule #0

009 | sub-M | Human-AI Collaboration As Documented Practice | What "we" actually means: shared notebook, divided labour, mutual correction. | claude_handoff.txt template

010 | fb-S | We Didn't Build A Chatbot | Why a chat-window-only AI is a regression from what we already do. | feedback_no_long_paste_in_chat.md

011 | sub-L | A Year On — What changed about how we live | Concrete day-to-day changes the system has produced. | weekly logbooks in Logbooks/

012 | fb-M | The Box On The Shelf — Our hardware is unglamorous on purpose | Jetson Orin Nano + tablet + ESP32s + speaker; commodity choices. | docs/JETSON_USER_GUIDE.md

013 | sub-M | Why SQLite — A schema you can fit in your head | One file, three core tables, no service to babysit. | docs/db_schema.md

014 | fb-S | We Use Markdown For Everything | Articles, logbooks, journals, handoff — plain text wins. | journal/, Logbooks/

015 | ig | Personal AI ≠ Personalised Ads | Three-line clarification. | thesis

016 | sub-M | The Three Surfaces — Carousel, Voice, Chat | Why we don't try to be one interface. | docs/voice-architecture.md, docs/carousel-architecture.md

017 | fb-M | The 03:00 Test — If it wakes you, it had better be right | Why we tune for false-positive over false-negative aggressively. | feedback_alerts_threshold.md

018 | sub-L | Against Smart Assistants — A polemic | The market's failure mode is loud, generic, forgetful. We went the other way. | thesis composite

019 | fb-S | Open Source Where It Helps, Closed Where It Doesn't | Pragmatic stack notes. | requirements.txt, package list

020 | ig | The Whole System Fits On A USB Stick | Punchy size brag. | du -sh on project

---

## B. Knowledge Graph — Anatomy (21–60)

021 | sub-L | Entities — The nouns of your life | Entity table walkthrough, type taxonomy, attribute attachment. | docs/db_schema.md, kb_search.py

022 | sub-M | Relationships — Directed verbs between nouns | Subject/object, predicate, temporal validity. | docs/relationship-modelling-spec.md

023 | sub-L | The Fact Table — Where prose meets schema | enhanced_facts: source, source_anchor, category, importance, confidence. | docs/db_schema.md, kb_insert.py [PCODE]

024 | sub-M | Attributes — The shape of an entity over time | When something is a fact vs an attribute; mutability, history. | attributes table, kb_search.py output

025 | fb-M | Aliases — The same thing by many names | Why we keep a separate alias table. | aliases table

026 | sub-M | The Audit Trigger — Every change leaves a trail | enhanced_facts_audit and the trg_facts_audit_update trigger. | CLAUDE.md rule 9

027 | fb-M | Don't Delete Facts, Supersede Them | Why old wrong facts stay in the record. | CLAUDE.md rule 9a

028 | sub-M | kb_search — Search that respects code-tokens | Case-sensitive exact-match pre-pass for tokens like PaN, AvW, C2. | tools/kb_search.py [PCODE]

029 | sub-M | kb_lint — Guard rails on relationship inserts | Forbidden predicates, person-subject checks, inverse-pair contradiction. | tools/kb_lint.py [PCODE], CLAUDE.md rule 15a

030 | fb-M | The Big Difference Incident | We recorded a charity collection backwards. The fix is a rule. | CLAUDE.md rule 15

031 | sub-M | Directional Read-Back — Saying it out loud before writing | Why we paraphrase a relationship in plain English before insert. | CLAUDE.md rule 15

032 | sub-L | No Single Source Of Truth — Why one Wikipedia tab isn't enough | The 2026-05-08 election-results incident and the rule it produced. | CLAUDE.md rule 0a

033 | sub-M | Timestamp And Source — On every state claim | the author's "I'm really bored with stale information" rule. | CLAUDE.md rule 0b

034 | fb-M | The Story Behind The Truncated Fact | The May council-tax incident — read the full body before flagging. | CLAUDE.md rule 0c

035 | sub-M | kb_search Top-3 Auto-Expand | Why we widened the preview window for perfect-score hits. | tools/kb_search.py

036 | sub-M | entity_check — Pre-flight before creating a new noun | Minimum three broad terms, alias hit detection. | tools/entity_check.py [PCODE]

037 | sub-M | fact_check — Pre-flight before stating a claim | Pull fact rows by content keyword across categories. | tools/fact_check.py [PCODE]

038 | fb-M | Storage Location Codes — A physical KB | Quadrants PaN/AvW/CaNE, drawer slots A4/C2, named boxes. | tools/storage_locations.py [PCODE]

039 | sub-M | kb_insert — One commit per write, no transactional creativity | Why each insert has its own connection. | tools/kb_insert.py [PCODE], CLAUDE.md rule 9

040 | sub-M | kb_dedup_facts — Merging near-identical entries | Hash buckets, content similarity, audit trail. | tools/kb_dedup_facts.py

041 | sub-M | kb_normalise_categories — Tidying the taxonomy | Why we let categories drift then sweep. | tools/kb_normalise_categories.py

042 | sub-M | kb_merge_duplicate_entities | When two "John"s are the same John. | tools/kb_merge_duplicate_entities.py

043 | sub-M | kb_conflict_scan — Catching contradictions in the corpus | Heuristics for inverse-pair predicates that disagree. | tools/kb_conflict_scan.py

044 | sub-M | kb_post_insert_audit — The trailing assertion | Background job that re-reads what was just inserted. | tools/kb_post_insert_audit.py

045 | sub-M | kb_clean_orphans | Entities with no facts, no relationships, no attributes. | tools/kb_clean_orphans.py

046 | fb-M | kb_incident_trace — Replaying a problem from the corpus | Pulling all rows around an incident date. | tools/kb_incident_trace.py

047 | sub-L | The Graph Gardener — Suggesting links you didn't notice | UI for low-effort triplet-confirmation passes. | tools/graph_gardener.py, kipple_app/routes/gardener.py

048 | sub-M | Graph Gaps — Detecting barren entities | Entities with attributes but no relationships. | tools/audit_graph_gaps.py

049 | sub-M | The 210→0 Barren Pass | A real session reducing barren-entity count. | recent commit ae0638f

050 | fb-M | What "Closeness" Used To Mean | We migrated closeness from a column to metadata. | tools/migrate_closeness_to_metadata.py

051 | sub-M | Relationship Classes — Bucketing predicates | Taxonomic grouping of relationship verbs. | tools/relation_classes.py

052 | sub-M | Temporal Tense On Attributes | future/past/now flags and the entity-context hook. | tools/temporal_tense.py

053 | sub-M | The Entity-Context Hook | What appears at the top of every Claude prompt. | tools/entity_context_hook.py [PCODE]

054 | sub-M | The Pre-Fetch Hook — Loading facts before the model speaks | Keyword fact prefetch architecture. | tools/journal_prefetch.py

055 | sub-M | kb_backfill_canonical_names | Backfilling normalised names without breaking aliases. | tools/kb_backfill_canonical_names.py

056 | sub-M | normalize_facts — Standardising prose on insert | Whitespace, smart-quote conversion, anchor casing. | tools/normalize_facts.py

057 | sub-M | backfill_context_keywords | Adding searchable keywords retroactively. | tools/backfill_context_keywords.py

058 | sub-M | dupe_scanner / orphan_scanner | Two scanners that earn their keep weekly. | tools/dupe_scanner.py, tools/orphan_scanner.py

059 | sub-M | The DB Health Check | What constitutes a healthy graph. | tools/db_health_check.py

060 | sub-L | The Graph Is The Product | Why a year of work points at the graph, not the chat. | docs/kb-architecture.md, feedback_link_entities.md

---

## C. The Carousel (61–95)

061 | sub-L | A Top-Trumps Deck For Daily Life — The carousel design philosophy | Uniform card schema, deck composability, eject rules. | docs/carousel-architecture.md, docs/carousel-v2-design.md

062 | sub-M | The Carousel V2 Design Doc — What we got wrong in V1 | Widget soup vs uniform-card deck. | docs/carousel-v2-design.md

063 | sub-M | Cards Don't Push Themselves — Intent layer 101 | Why timer-driven carousels become unbearable. | feedback_carousel_trigger.md

064 | sub-M | Eject Rules As Part Of The Card | Cards own their own death. | docs/carousel-architecture.md

065 | sub-M | The Dining-Room-Light Trigger | The single hardware event that opens the bedtime window. | feedback_carousel_trigger.md

066 | sub-M | Carousel V2 Shadow Mode — Comparing two carousels live | How we A/B'd before switching. | tools/carousel_v2_shadow_summary.py

067 | sub-M | carouselctl — The carousel's command-line | Push, eject, list, replay. | tools/carouselctl.py [PCODE], docs/carouselctl.md

068 | sub-M | The Ghost Card Incident, 2026-05-11 | A card that wouldn't eject. The fix. | docs/carousel-ghost-card-2026-05-11.md

069 | fb-M | "Refill Vinegar: Done" — How ack speech works | Terse verb-noun ack on double-click. | feedback_ack_speech_terse.md

070 | sub-M | carousel_pulse — The breathing-light service | Hue light pulses while a card is live. | tools/carousel_pulse.py [PCODE], carousel-pulse.service

071 | fb-M | The Smart-Button Sandwich | Carousel a/c/k SmartThings buttons on Z2M. | feedback_carousel_buttons_identity.md

072 | sub-M | One Channel, One Mention | The rule that made the carousel quiet enough to live with. | feedback_no_nagging.md

073 | sub-M | Replaying Yesterday's Deck — A surprisingly good daily review | What we use the replay for. | tools/carousel_cards.py

074 | sub-M | The Lap-Eject Pattern | Exclusive-window cards that go stale at midnight. | upcoming deadline 2026-05-18

075 | sub-M | The Monthly-Bill Carousel — Generic pattern, replacing one-offs | What replaced the hardcoded council-tax block. | fact about generic monthly-bill pattern, 2026-05-14

076 | sub-M | Speech On Eject — When to say something, when not | The terse-ack rule and its exceptions. | feedback_ack_speech_terse.md

077 | sub-M | DUE NOW vs On The Carousel — Two different states | A subtle source-of-truth distinction. | feedback_carousel_trigger.md

078 | sub-M | The Carousel Doctor | Self-diagnosis when cards stop appearing. | tools/carousel_doctor.py

079 | sub-M | morning_carousel_test — A repeatable smoke test | docs/morning-carousel-test-procedure.md walkthrough. | docs/morning-carousel-test-procedure.md

080 | sub-M | The Inkling Button — Reverse-direction input to the system | Phone-side speech-to-carousel-note. | session 67d633b commit

081 | fb-M | The SAFETY Tripwire | Carousel push pattern for safety-critical hazards. | recent commit c3c0966

082 | sub-M | Carousel MQTT Subscriber | The service that listens for Z2M button presses. | carousel-mqtt-subscriber.service

083 | sub-M | test_carousel_cards_cross_day | Edge-case test for cards spanning midnight. | tools/test_carousel_cards_cross_day.py

084 | sub-M | The Carousel As Top-Of-Funnel | Everything important earns a card; cards earn voice; voice earns nothing else. | composite

085 | sub-M | Carousel Slot Templates — Stamping cards from data | How a stub becomes a card. | project_carousel_trumps_model.md

086 | fb-M | The Chickpea Card | A real card, a real fix. | tools/inject_carousel_chickpea_fix_2026_04_25.py

087 | sub-M | Cards For Other People — Tagged context-only events | [name], [organisation] prefixes and how they're filtered. | feedback_calendar_tag_ownership.md

088 | sub-M | The Provenance Dot — Confidence-coded card markers | How we colour-code who suggested a card. | feature spec

089 | sub-M | Card TTLs — Why every card has a maximum age | The TTL field and what happens at expiry. | docs/carousel-architecture.md

090 | sub-M | Eject Cascade — One ack, many ejects | When acknowledging A also ejects B and C. | docs/carousel-architecture.md

091 | sub-M | Carousel Logs — Forensics on a missed card | Where we look when a card didn't appear. | recent debug sessions

092 | sub-M | Carousel On Mobile — The phone bubble fallback | When the kitchen tablet isn't where you are. | kipple_speak.py [PCODE]

093 | fb-M | The Stream Deck Bridge | Carousel via tactile buttons. | tools/streamdeck_poc.py, tools/stream_deck_bridge.py

094 | sub-M | What We Won't Put On The Carousel | The negative space of the deck. | feedback_no_nagging.md

095 | sub-L | The Carousel After 12 Months — What stayed, what got cut | A retrospective. | composite

---

## D. Architecture Doc Walks — One Per docs/*-architecture.md (96–130)

096 | sub-L | The Agent Relay — How agents talk to each other | docs/agent-relay-architecture.md deep-dive. | docs/agent-relay-architecture.md

097 | sub-L | The Alert Pipeline | docs/alert-architecture.md deep-dive. | docs/alert-architecture.md

098 | sub-L | The Arduino Side Of Things | docs/arduino-architecture.md deep-dive. | docs/arduino-architecture.md

099 | sub-L | The Astrolabe — Live status panel framework | docs/astrolabe-architecture.md walkthrough. | docs/astrolabe-architecture.md, recent commit 67d633b

100 | sub-L | The Automation Engine — Rules over events | docs/automation-engine-architecture.md. | docs/automation-engine-architecture.md

101 | sub-L | The Breathing Light | docs/breathing-light-architecture.md. | docs/breathing-light-architecture.md

102 | sub-L | The Carousel Architecture | docs/carousel-architecture.md. | docs/carousel-architecture.md

103 | sub-L | Claude Discipline — Rules a coding model has to follow | docs/claude-discipline-architecture.md. | docs/claude-discipline-architecture.md

104 | sub-L | Claude Terminal — Mobile web on port 3131 | docs/claude-terminal-architecture.md. | docs/claude-terminal-architecture.md

105 | sub-L | Component Inventory As Architecture | docs/component-inventory-architecture.md. | docs/component-inventory-architecture.md

106 | sub-L | Disk Sentinel — Watching the SSDs | docs/disk-sentinel-architecture.md. | docs/disk-sentinel-architecture.md

107 | sub-L | Email Send Path — SMTP over OAuth2 | docs/email-send-architecture.md. | docs/email-send-architecture.md

108 | sub-L | The Gantt Chart As Source-Of-Plans | docs/gantt-chart-architecture.md. | docs/gantt-chart-architecture.md

109 | sub-L | Git Sentinel — Catching un-pushed work | docs/git-sentinel-architecture.md. | docs/git-sentinel-architecture.md

110 | sub-L | The Graph Gardener Architecture | docs/graph-gardener-architecture.md. | docs/graph-gardener-architecture.md

111 | sub-L | The Heliograph — Signalling between Orinoco services | docs/heliograph-architecture.md. | docs/heliograph-architecture.md

112 | sub-L | Home Automations | docs/home-automations-architecture.md. | docs/home-automations-architecture.md

113 | sub-L | House Power — Tracking the meter | docs/house-power-architecture.md. | docs/house-power-architecture.md

114 | sub-L | Ingestion — Pulling external feeds into the KB | docs/ingestion-architecture.md. | docs/ingestion-architecture.md, docs/ingestion.md

115 | sub-L | The Journal — Daily prose, machine-assembled | docs/journal-architecture.md. | docs/journal-architecture.md

116 | sub-L | Journeys — Recurring trips as templates | docs/journey-architecture.md. | docs/journey-architecture.md

117 | sub-L | The KB Architecture | docs/kb-architecture.md. | docs/kb-architecture.md

118 | sub-L | LocalTuya — Liberating cheap smart-plugs | docs/localtuya-architecture.md, docs/project_tuya_liberation.md. | docs/localtuya-architecture.md

119 | sub-L | Mileage — Auto-deriving expense claims | docs/mileage-architecture.md. | docs/mileage-architecture.md

120 | sub-L | Missy Clean — Pet welfare automation | docs/missy-clean-architecture.md. | docs/missy-clean-architecture.md

121 | sub-L | Mum's Journals — Family archive ingestion | docs/mums-journals-architecture.md. | docs/mums-journals-architecture.md

122 | sub-L | Orinoco Network — The home server's connectivity | docs/orinoco-network-architecture.md. | docs/orinoco-network-architecture.md

123 | sub-L | Packing — Pre-trip checklists with memory | docs/packing-architecture.md. | docs/packing-architecture.md

124 | sub-L | The Recovery System | docs/recovery-architecture.md. | docs/recovery-architecture.md

125 | sub-L | Room Audio Duck — Ducking music for speech | docs/room-audio-duck-architecture.md. | docs/room-audio-duck-architecture.md

126 | sub-L | Session Architecture — One conversation, many turns | docs/session-architecture.md. | docs/session-architecture.md

127 | sub-L | The Switchboard | docs/switchboard-architecture.md. | docs/switchboard-architecture.md

128 | sub-L | Voice — From microphone to action | docs/voice-architecture.md. | docs/voice-architecture.md

129 | sub-L | VSCode + Claude — The IDE integration | docs/vscode-claude-architecture.md. | docs/vscode-claude-architecture.md

130 | sub-L | Weather — Forecast, observation, nowcast | docs/weather-architecture.md. | docs/weather-architecture.md

---

## E. The Multi-Agent Fleet (131–155)

131 | sub-L | Claude Code, Codex, Claude Desktop — Three brains, one notebook | How we use each, when we pick which. | claude_inbox.txt, codex_inbox.txt

132 | sub-M | The Inbox Files — Asynchronous agent IPC | claude_inbox / codex_inbox / phone_inbox as a transport. | tools/inbox_writer.py

133 | sub-M | ask_codex — Calling Codex from Claude | The shape of a delegated query. | tools/ask_codex.py [PCODE]

134 | sub-M | codex_inbox Backups | Why we snapshot the inbox before every handoff. | the .bak-NNNNN files in repo root

135 | sub-M | The Handoff File | Mandatory template, atomic writes via handoff_write.py. | CLAUDE.md handoff section, tools/handoff_write.py [PCODE]

136 | sub-M | Sidenote — A side-channel that doesn't interrupt | kinds: cmd, idea, journal, fyi, stop. | CLAUDE.md sidenote section, tools/sidenote.py [PCODE]

137 | sub-M | Urgent Sidenote — When "stop" really means stop | The persist-until-ack rule. | tools/sidenote_urgent.py

138 | sub-M | kipple-codex — The /kipple-codex skill | Two-way check between Claude and Codex. | skill manifest

139 | sub-M | Why Three Models — The case for diversity of advice | When agreement is consensus, when disagreement is information. | composite

140 | fb-M | Claude Asks, Codex Tells, Desktop Reviews | An informal protocol that emerged. | observed pattern

141 | sub-M | The Inbox Tail Watcher | Background process that surfaces new lines. | tools/voice_dispatch_tail.py

142 | sub-M | The Compact Nudge | Telling a long session to summarise itself. | tools/compact_nudge.py

143 | sub-M | Context Size Monitor | Watching how much we've fed the model. | tools/context_size_monitor.py

144 | sub-M | The Session Architecture | One transcript per session, durable on disk. | docs/session-architecture.md

145 | sub-M | Crash Recovery From Transcripts | When the session dies, what we read first. | reference_transcript_recovery.md

146 | sub-M | The Claude-Terminal Phantom Olympics | A Whisper hallucination two-word phantom incident. | facts 15817+15818

147 | sub-M | Don't Re-Diagnose Shipped Fixes | The handoff trust rule. | feedback_dont_rediagnose_shipped.md

148 | sub-M | Restart The Correct Service | claude-terminal vs kipple-unified — different tools, different restarts. | feedback_restart_correct_service.md

149 | sub-M | Claude On Windows — Why we ssh to Orinoco | docs/orinoco-route-repair-sudoers.md context. | composite

150 | sub-M | VSCode Fact-Check Integration | tools/inject_vscode_fact_check_2026_04_23.py. | tools/inject_vscode_fact_check_2026_04_23.py

151 | sub-M | The Anthropic Probe | tools/anthropic_probe.py — testing the API. | tools/anthropic_probe.py

152 | sub-M | The Codex Probe | tools/codex_probe.py. | tools/codex_probe.py

153 | sub-M | API OAuth Probe | tools/api_oauth_probe.py. | tools/api_oauth_probe.py

154 | sub-M | The Codex Usage Monitor | Watching token spend. | tools/codex_usage_monitor.py

155 | sub-L | Why "We" — On writing as a partnership | The first-person-plural rule and its reasons. | this index

---

## F. AI Discipline & Pitfalls (156–195)

156 | sub-L | Verify Before Speaking — The core failure mode | 18+ corrections; the rule and its consequences. | feedback_verify_before_speaking.md

157 | sub-M | The Stop-After-The-Substantive-Answer Rule | The patio-door flourish incident. | CLAUDE.md, 2026-05-17 09:33 BST

158 | sub-M | flourish_search — Only sanctioned embellishment | The curated corpus and the audit log. | tools/flourish_search.py [PCODE]

159 | sub-M | The Volatile-Wikipedia Election Incident | What two embarrassing Facebook posts taught the system. | CLAUDE.md rule 0a, 2026-05-08

160 | sub-M | The Artemis Tonight-vs-Tomorrow Incident | Why we now run `date` before any temporal claim. | MEMORY.md mandatory time rule

161 | sub-M | Stale Information Is A Bug | The 2026-05-08 morning composite of stale carousel/alert/location. | CLAUDE.md rule 0b

162 | sub-M | The Truncated-Preview Trap | Read the full fact before flagging a discrepancy. | CLAUDE.md rule 0c

163 | sub-M | Embellishment Must Be Factual | A welcome flourish, a wrong flourish, the difference. | feedback_embellishment_must_be_factual.md

164 | sub-M | No Scene Dressing | Why a clear sentence beats a homely paragraph. | feedback_no_scene_dressing.md

165 | sub-M | Request-Coverage Diff — Composing for others | Map each line of the draft to a literal ask. | feedback_request_coverage_diff.md

166 | sub-M | The Email Sender-Vicinity Rule | Why we sweep the recipient's recent threads before composing. | feedback_sweep_sender_inbox_before_reply.md

167 | sub-M | The Yellow-Banner Draft Trap | Why gmail_send.py exists and Gmail API drafts.send doesn't. | feedback_gmail_draft_path.md, CLAUDE.md Gmail section

168 | sub-M | Send Codeword Flow — Authorising a send | One codeword, one send. | CLAUDE.md send-codeword section

169 | sub-M | Search Strategy — Single distinctive terms first | Why multi-word AND/OR combos under-perform. | feedback_search_strategy.md

170 | sub-M | Don't Fill Gaps In Dictation | Two slips: Kohinoor, Oxford-as-London. | feedback_dictation_gap_hallucination.md

171 | sub-M | Check KB Before Calling A Word A Stray | The alias/attribute resolution rule. | feedback_check_kb_before_calling_stray.md

172 | sub-M | Possessive-Pronouns Check | Why "the author's keys" and "The Compatriot's keys" need separating. | feedback_possessive_pronouns.md

173 | sub-M | Disambiguation — State And Ask, Never Guess | Multiple people share a name. | feedback_disambiguation_ask.md

174 | sub-M | Med-Timing — Never Assume Safe | Health-critical. Always check the tickoff time. | feedback_med_timing_check.md

175 | sub-M | Don't Nag — One mention per real context | The cardinal sin of life-management software. | feedback_no_nagging.md

176 | sub-M | The Goodbye Check | No "go enjoy" with unticked items. | feedback_check_carousel_before_goodbye.md

177 | sub-M | Auto-Clear TODOs On Acknowledgement | When the author says he did it, mark it done. | feedback_auto_clear_todos.md

178 | sub-M | Auto-Verify Speech Via Mic | Don't ask "did you hear it?", read the Vosk transcript. | feedback_verify_speech_via_mic.md

179 | sub-M | Don't Hold Back Drafts To Avoid Auto-Voice | Show the user the draft when asked, don't hide it. | feedback_dont_withhold_content.md

180 | sub-M | No URLs In Speech | URLs go in the bubble, never the spoken text. | feedback_no_urls_in_speech.md

181 | sub-M | Speech-Friendly Output — No IDs, Expand Abbreviations | TTS, KB, HA, OAuth all spelled out. | feedback_speech_friendly_no_ids.md

182 | sub-M | Carousel Ack Speech Is Terse | "verb noun: done" not "noted". | feedback_ack_speech_terse.md

183 | sub-M | Do Obvious Diagnostic Steps Without Asking | Chain read-only checks. | feedback_do_obvious_diagnostic_steps.md

184 | sub-M | Fix The Bug, Not The Feature | When the input data is bad, don't blame the code. | feedback_fix_the_bug_not_the_feature.md

185 | sub-M | Trigger + Elapsed, Not Clock Brackets | Outlier-resistant time windows. | feedback_trigger_plus_elapsed.md

186 | sub-M | KB Narrative Style — Prose, Not Nuggets | Why descriptive paragraphs beat atomised facts. | feedback_kb_narrative_style.md

187 | sub-M | Speech Delta, Not Topic | "The Compatriot replied re the form" beats "letter about the form". | feedback_speech_delta_not_topic.md

188 | sub-M | Custom Tools Over Write For State Files | The handoff_write.py rationale. | feedback_custom_tools_over_write.md

189 | sub-M | Tease Out Detail When The User Shares A Memory | Don't rush to the next topic. | feedback_tease_out_detail.md

190 | sub-M | Check Transcript Logs On Session Start | When the handoff is thin. | feedback_check_transcripts.md

191 | sub-M | Broad Sweep For Recaps | "What did I do this week" needs all events, not keywords. | feedback_broad_sweep_for_recaps.md

192 | sub-M | Pin Useful URLs To The Links Popup | Phone-side discoverability. | feedback_pin_useful_urls.md

193 | sub-M | Offer Watchlist Additions Proactively | When the user is waiting for an email. | feedback_offer_watchlist.md

194 | sub-M | Web-Search New Places On Entity Creation | Address, hours, status. | feedback_websearch_new_places.md

195 | sub-L | The Discipline Of Slowing Down At The Right Moment | Composite of all the above — speed at the wrong thing is worse than slow at the right one. | composite

196 | sub-M | Memory Is Wallpaper — Why we hard-code rules instead | MEMORY.md rule #0 in detail. | MEMORY.md

197 | sub-M | The Pre-Commit Hook | Architecture doc + logbook entry required for python commits. | githooks/pre-commit, CLAUDE.md rule 1a

198 | sub-M | pre_commit_review — What the script actually checks | Walkthrough. | tools/pre_commit_review.py [PCODE]

199 | sub-M | The Arch-Doc Reminder Hook | When you change a file, get reminded its arch doc may rot. | tools/arch_doc_hook.py [PCODE]

200 | sub-M | arch_doc_staleness_check | Periodic verification that docs still match code. | tools/arch_doc_staleness_check.py

---

## G. Components — Microscopes With Pseudocode (201–280)

201 | sub-M | glance.py — The pre-response context pull | docs/voice-architecture.md context. | tools/glance.py [PCODE], CLAUDE.md glance section

202 | sub-M | context_gather.py — Pre-task material sweep | What the rule "context gather before any task" actually does. | tools/context_gather.py [PCODE]

203 | sub-M | context_brief.py — The rolling brief | Short, structured, near-now. | tools/context_brief.py [PCODE]

204 | sub-M | gmail_search.py — Our own Gmail search | Why we don't use MCP. | tools/gmail_search.py [PCODE]

205 | sub-M | gmail_send.py — SMTP+OAuth, no banner | The send path. | tools/gmail_send.py [PCODE]

206 | sub-M | gmail_draft.py — API drafts.create | The review path. | tools/gmail_draft.py [PCODE]

207 | sub-M | gmail_update_draft.py — Editing a pending draft | The iteration path. | tools/gmail_update_draft.py [PCODE]

208 | sub-M | sender_vicinity.py — The pre-compose sweep | 15-min marker, freshness check. | tools/sender_vicinity.py [PCODE]

209 | sub-M | gcal_search.py — Calendar search that works | Why MCP calendar search isn't enough. | tools/gcal_search.py [PCODE]

210 | sub-M | gcal_journey.py — Journey-aware calendar | Recurring trips as templates. | tools/gcal_journey.py [PCODE]

211 | sub-M | gcal_reauth.py / google_reauth.py | Token refresh for OAuth. | tools/gcal_reauth.py, tools/google_reauth.py

212 | sub-M | weather_obs.py — Local observation | Sourcing weather from a near sensor. | tools/weather_obs.py [PCODE]

213 | sub-M | rain_nowcast.py — Rain in the next 6 hours | Met-Office or equivalent. | tools/rain_nowcast.py [PCODE]

214 | sub-M | weather_calendar.py — Forecasts as calendar events | The weather-change-tomorrow-02:00 pattern. | tools/weather_calendar.py [PCODE]

215 | sub-M | bird_flu_monitor.py | Defra surveillance feed scrape. | tools/bird_flu_monitor.py [PCODE]

216 | sub-M | followup_reminder.py — The dedicated reminder CLI | Why not just a calendar event. | tools/followup_reminder.py [PCODE], CLAUDE.md follow-up section

217 | sub-M | tracker.py — Where we are right now | Phone location, breadcrumbs, ground truth. | tools/tracker.py [PCODE]

218 | sub-M | location.py / location_match.py / location_pin.py | The location stack. | tools/location.py [PCODE]

219 | sub-M | route_map.py / route_map_3d.py / daily_route_map.py | Map renderers. | tools/route_map.py

220 | sub-M | journey_monitor.py — Watching a trip in progress | The departure-aware monitor. | tools/journey_monitor.py [PCODE]

221 | sub-M | retro_journey_replay.py | Replaying a journey after the fact. | tools/retro_journey_replay.py

222 | sub-M | journey_template_sentinel.py | Detecting drift from a template. | tools/journey_template_sentinel.py

223 | sub-M | timeline_parser.py / timeline_dedupe.py / timeline_to_calendar.py | Google Timeline ingestion. | tools/timeline_parser.py [PCODE]

224 | sub-M | morning_brief.py / morning_brief_warmer.py | The pre-morning warmup. | tools/morning_brief.py [PCODE]

225 | sub-M | bootstrap_state.py | Initial state file generation. | tools/bootstrap_state.py

226 | sub-M | state_io.py — Atomic state file writes | The pattern we reuse everywhere. | tools/state_io.py [PCODE]

227 | sub-M | trigger_eval.py — Evaluating a card's trigger | Why some cards wait. | tools/trigger_eval.py [PCODE]

228 | sub-M | dependency_scheduler.py | Tasks with prerequisites. | tools/dependency_scheduler.py [PCODE]

229 | sub-M | task_timer.py — Ad-hoc timed reminders | The full-date rule for one-shots. | tools/task_timer.py [PCODE]

230 | sub-M | kipple_alerts.py — The alert dispatcher | The composite of everything above. | tools/kipple_alerts.py [PCODE]

231 | sub-M | kipple_announce.py | One-shot announcement helper. | tools/kipple_announce.py

232 | sub-M | kipple_say.py — Speech to cavern/phone/both/alexa | Destination routing. | tools/kipple_say.py [PCODE]

233 | sub-M | kipple_speak.py — The MacroDroid speak helper | Modes: sil/mp3/nat/del/stop. | tools/kipple_speak.py [PCODE]

234 | sub-M | output_router.py — Where speech actually goes | The route-decision tree. | tools/output_router.py [PCODE]

235 | sub-M | tts_route.py / tts_voice_settings.py / tts_playground.py | The TTS stack. | tools/tts_route.py [PCODE]

236 | sub-M | voice_wake.py — Wake-word listener | What we listen for, what we ignore. | tools/voice_wake.py [PCODE], voice-wake.service

237 | sub-M | voice_session.py | One conversation, end-to-end. | tools/voice_session.py [PCODE]

238 | sub-M | voice_dispatch_log.py — Speech outbound log | Forensics on what was said. | tools/voice_dispatch_log.py [PCODE]

239 | sub-M | voice_message_ledger.py — The Vosk-confirmed log | docs/voice-message-ledger.md. | tools/voice_message_ledger.py [PCODE], docs/voice-message-ledger.md

240 | sub-M | voice_audio_cache.py / voice_audio_cleanup.py | The MP3 cache lifecycle. | tools/voice_audio_cache.py

241 | sub-M | voice_path_watchdog.py | Watching for silent failures. | tools/voice_path_watchdog.py [PCODE]

242 | sub-M | voice_conv_cli.py | CLI for inspecting recent conversations. | tools/voice_conv_cli.py

243 | sub-M | voice_repeat_last.py | "Say that again". | tools/voice_repeat_last.py

244 | sub-M | speech_gate_test.py / speech_path_probe.py / speech_lock.py | The speech-path smoke kit. | tools/speech_gate_test.py [PCODE]

245 | sub-M | speech_away.py — Location-aware downgrade | When to bubble-only. | tools/speech_away.py [PCODE]

246 | sub-M | room_speech_log.py / room_speech_monitor.py / room_silence.py | Per-room speech tracking. | tools/room_speech_log.py [PCODE]

247 | sub-M | room_audio_duck.py — Ducking music for an alert | docs/room-audio-duck-architecture.md. | tools/room_audio_duck.py [PCODE]

248 | sub-M | active_speaker.py | Knowing which speaker is hot. | tools/active_speaker.py

249 | sub-M | duck_events.py | Calendar-driven audio ducking. | tools/duck_events.py [PCODE]

250 | sub-M | snowball_whisper.py / recognize_track.py | Music ID. | tools/snowball_whisper.py

251 | sub-M | last_spoken.py | When did we last say anything to a destination. | tools/last_spoken.py

252 | sub-M | front_door_watcher.py / front_door_ack.py | Person-in-zones speech + alcove flash. | tools/front_door_watcher.py [PCODE], front-door-watcher.service

253 | sub-M | test_front_door_watcher_vision_gate.py | Vision-gate regression. | tools/test_front_door_watcher_vision_gate.py

254 | sub-M | ha_camera_snapshot.py | On-demand camera frame grab. | tools/ha_camera_snapshot.py [PCODE]

255 | sub-M | face_recog_smoke.py | Smoke-test for face recognition. | tools/face_recog_smoke.py

256 | sub-M | uvc_exposure_probe.py | Webcam exposure tuning. | tools/uvc_exposure_probe.py

257 | sub-M | ha_sync.py / ha_sync_watcher.py | Home Assistant state mirror. | tools/ha_sync.py [PCODE]

258 | sub-M | aviary_sun_daemon.py — Polly-Lumens | Congo-basin equatorial sun simulation. | tools/aviary_sun_daemon.py [PCODE], kipple-aviary-sun.service

259 | sub-M | ambient_light.py | Whole-house ambient light state. | tools/ambient_light.py [PCODE]

260 | sub-M | astrolabe.py / astrolabe_web | The live status panel. | tools/astrolabe.py [PCODE]

261 | sub-M | switchboard_voice_command.py / switchboard_e2e_test.py | docs/kipple-switchboard.md. | tools/switchboard_voice_command.py [PCODE]

262 | sub-M | bt_scanner.py — Bluetooth presence | Who's home. | tools/bt_scanner.py [PCODE]

263 | sub-M | devices_sentinel.py — Battery/firmware health | Per-device monitoring. | tools/devices_sentinel.py [PCODE]

264 | sub-M | disk_sentinel.py + classifier | docs/disk-sentinel-design.md. | tools/disk_sentinel.py [PCODE]

265 | sub-M | disk_sentinel_gate.py / disk_sentinel_rules.py | The decision layer. | tools/disk_sentinel_gate.py [PCODE]

266 | sub-M | git_sentinel.py — Unpushed-work warner | docs/git-sentinel-architecture.md. | tools/git_sentinel.py [PCODE]

267 | sub-M | missy_clean_sentinel.py | The cage-clean schedule sentinel. | tools/missy_clean_sentinel.py [PCODE]

268 | sub-M | orinoco_status.py / orinoco_route_guard.py / orinoco_guest_greeter.py | Host watchers. | tools/orinoco_status.py [PCODE]

269 | sub-M | victory_scanner.py / victory_scanner_cron.py / victories_speech_backfill.py | Acknowledging wins. | tools/victory_scanner.py [PCODE]

270 | sub-M | important_emails.py / kb_extract_artists_fb_events.py / scrape_fb_events.py | The ingestion fleet. | tools/important_emails.py [PCODE]

271 | sub-M | email_watchlist.py / watchlist_enricher.py / reply_watcher.py | Watchlist primitives. | tools/email_watchlist.py [PCODE]

272 | sub-M | important_emails_state.json — The watchlist state | What gets surfaced and why. | tools/important_emails_state.json

273 | sub-M | smart_fact_capture.py | Auto-promoting good chat lines to facts. | tools/smart_fact_capture.py [PCODE]

274 | sub-M | conversational_knowledge.py | Knowledge graph reads in flow. | tools/conversational_knowledge.py

275 | sub-M | resolve_audit_findings.py — The /kipple-resolve loop | Interactive review. | tools/resolve_audit_findings.py [PCODE]

276 | sub-M | journal_prep.py / journal_pager.py / journal_recall.py / journal_quality.py | The journal stack. | tools/journal_prep.py [PCODE]

277 | sub-M | journal_kb_sweep.py / journal_fragment.py / journal_status.py | Journal hygiene. | tools/journal_kb_sweep.py [PCODE]

278 | sub-M | journal_gdoc.py — Journal to Google Docs | Re-used by social_articles_doc.py. | tools/journal_gdoc.py [PCODE]

279 | sub-M | backup_kb_orinoco.py / backup_mums_journals.py / verify_mums_journals_backup.py | The backup stack. | tools/backup_kb_orinoco.py [PCODE]

280 | sub-M | inventory_orinoco.py / kb_metadata_enricher.py | Sweep utilities. | tools/inventory_orinoco.py [PCODE]

---

## H. Hardware & Physical (281–320)

281 | sub-M | The Jetson Orin Nano — Why we picked it | docs/JETSON_USER_GUIDE.md context. | docs/JETSON_USER_GUIDE.md

282 | sub-M | Plantronics + USB Audio — The Cavern speaker chain | Why a wired chain beats Bluetooth here. | KB Plantronics entity facts

283 | sub-M | Xr12/Xr16 — Mixer control over IP | tools/xr12_control.py et al. | tools/xr12_control.py [PCODE]

284 | sub-M | The Cavern Air Scrubber Wiring | Three speeds on the internal board. | fact about P4/P5 motor coils

285 | sub-M | The Logic Probe Project | ESP8266 + OLED, 10kHz default sample. | KB Logic-Probe project facts

286 | sub-M | ChatGPET — Matrix keyboard tester | TPIC6B595 + 4066 + 4067 + OLED. | KB ChatGPET project facts

287 | sub-M | The Z2M Network — SmartThings + Aqara buttons | What's on the mesh. | KB Z2M entity facts

288 | sub-M | Hue Bulbs — Carousel-pulse breathing | Why we don't use Hue scenes for buttons. | feedback_carousel_buttons_identity.md

289 | sub-M | The Tablet On The Shelf | What it runs, how it's mounted, what fails. | composite

290 | sub-M | Front-Door Camera — Nest + Frigate | Person-in-zone events. | front-door-watcher.service, docs/extractors.md

291 | sub-M | Frigate On JP5 — The container compat trap | docs/extended_rules.md container rule. | CLAUDE.md rule 16

292 | sub-M | Frigate Network Mode Host — The mosquitto reach trap | Why bridge networking failed. | CLAUDE.md rule 17

293 | sub-M | The Path-B Face-Recognition Announcer | Local replacement for the cloud-Sonnet round-trip. | recent logbook fact, 2026-05-17

294 | sub-M | LocalTuya Liberation | Why we left the Tuya cloud. | docs/project_tuya_liberation.md

295 | sub-M | The GD Radio Dial Project | tools/inject_gd_radio_* helpers. | docs/gd_radio_parts_list.md

296 | sub-M | The Fan Hausing Project | Dust extractor for parrot dander. | tools/inject_fan_hausing_gantt.py

297 | sub-M | Polly-Lumens — Aviary equatorial sun | LED strip + ramp profile. | tools/aviary_sun_daemon.py [PCODE]

298 | sub-M | The Stream Deck POC | tools/streamdeck_poc.py + paged decks. | recent commit 4f003cb

299 | sub-M | Workbench Server | tools/workbench_server.py — bench-side helper. | tools/workbench_server.py [PCODE]

300 | sub-M | Bin Pickups, Storage Quadrants | tools/storage_locations.py grid. | tools/storage_locations.py [PCODE]

---

## I. Email, Inbox, Postageist (301–325)

301 | sub-L | Postageist — A daily brief over your inbox | What the brief covers and how it's assembled. | postageist/ adjacent project

302 | sub-M | Why We Don't Use MCP Gmail | CLAUDE.md Gmail section in detail. | CLAUDE.md

303 | sub-M | Watchlists — Things we're waiting on | [WATCHLIST:] tags in briefs. | feedback_offer_watchlist.md

304 | sub-M | The Inbox-To-Jobs-List Matcher | CLAUDE.md inbox-matcher rule. | CLAUDE.md

305 | sub-M | Match-Strength — Why we require two signals | The 2026-05-13 Art House mail-server incident. | CLAUDE.md rule on match strength

306 | sub-M | Sender-Vicinity — The 15-min freshness window | The hard refuse in gmail_send.py. | feedback_sweep_sender_inbox_before_reply.md

307 | sub-M | The Send Codeword | One word, one send, no banner. | CLAUDE.md

308 | sub-M | gmail_draft vs Gmail-API drafts.send | The yellow-banner subtlety. | feedback_gmail_draft_path.md

309 | sub-M | Important-Emails State File | Where the watchlist actually lives. | tools/important_emails_state.json

310 | sub-M | Email Discipline | One paragraph rule, several sub-rules. | feedback_email_discipline.md

311 | sub-M | Auto-Reply Watcher | tools/reply_watcher.py — closing the loop. | tools/reply_watcher.py [PCODE]

312 | sub-M | Watchlist Enricher | tools/watchlist_enricher.py. | tools/watchlist_enricher.py

313 | sub-M | The Email-Send Architecture | docs/email-send-architecture.md. | docs/email-send-architecture.md

314 | sub-M | Gmail Archaeology — Condensed | tools/condense_gmail_archaeology.py. | tools/condense_gmail_archaeology.py

315 | sub-M | Download-Gmail-Attachments | tools/download_gmail_attachments.py. | tools/download_gmail_attachments.py

316 | sub-M | The Amazon Monitor | Order-status scrape. | tools/amazon_monitor.py [PCODE]

317 | sub-M | Temu Order Scraper | tools/scrape_temu_orders.py. | tools/scrape_temu_orders.py

318 | sub-M | Inject_aliexpress_* — Capturing online purchases | The shape of a purchase fact. | tools/inject_aliexpress_*.py

319 | sub-M | Enrich Purchase Specs | tools/enrich_purchase_specs.py. | tools/enrich_purchase_specs.py [PCODE]

320 | sub-M | Receipts As Facts | category=purchase, anchor=order_id. | CLAUDE.md rule 8

321 | sub-M | Xero Mileage / Auth | tools/xero_auth.py + tools/xero_mileage.py. | tools/xero_auth.py [PCODE]

322 | sub-M | Mileage From Journeys | docs/mileage-architecture.md. | docs/mileage-architecture.md

323 | sub-M | Co-working Project Logbook | tools/inject_coworking_project_2026_04_08.py. | tools/inject_coworking_project_2026_04_08.py

324 | sub-M | NDAC Update Capture | tools/inject_ndac_update_2026_04_16.py. | tools/inject_ndac_update_2026_04_16.py

325 | sub-M | The Police-Journal Fact | tools/inject_journal_2026_05_01_police.py. | tools/inject_journal_2026_05_01_police.py

---

## J. Logbooks, Journals, Family Archive (326–360)

326 | sub-L | The Logbook Habit — Why we write things down twice | Logbooks vs journals vs facts. | Logbooks/ folder

327 | sub-M | Weekly Logbooks — One file per week | The structure and rotation. | Logbooks/LOGBOOK-YYYY-Www_*.md

328 | sub-M | Logbook Consolidator | tools/logbook_consolidator.py. | tools/logbook_consolidator.py [PCODE]

329 | sub-M | The Journal Architecture | docs/journal-architecture.md. | docs/journal-architecture.md

330 | sub-M | journal_prep — Pre-writing the prompt | What the model sees. | tools/journal_prep.py [PCODE]

331 | sub-M | journal_pager — Processing one page at a time | Why we paginate. | tools/journal_pager.py [PCODE]

332 | sub-M | journal_quality — Self-assessment | A quality score on the day's prose. | tools/journal_quality.py

333 | sub-M | journal_recall — Finding a memory | The retrieval side. | tools/journal_recall.py [PCODE]

334 | sub-M | journal_kb_sweep — Promoting prose to facts | Where good lines go. | tools/journal_kb_sweep.py

335 | sub-M | journal_gdoc — Mirroring to Google Docs | The pattern this very index reuses. | tools/journal_gdoc.py [PCODE]

336 | sub-M | Daily Briefs — journal/YYYY-MM-DD_brief.md | What's in a brief. | journal/*_brief.md

337 | sub-M | Mum's Journals — Family archive ingestion | docs/mums-journals-architecture.md. | docs/mums-journals-architecture.md

338 | sub-M | backup_mums_journals + verify | Belt-and-braces backup. | tools/backup_mums_journals.py [PCODE]

339 | sub-L | Why Personal Archive Matters | The dementia-adjacent case for continuity. | composite — generalities only

340 | sub-M | Family Biography Facts | category=biography in the KB. | enhanced_facts WHERE category='biography'

341 | sub-M | The Inventory Sweep | tools/inventory_orinoco.py. | tools/inventory_orinoco.py

342 | sub-M | Storage Quadrant Aliases | 41 codes, all aliased to storage_quadrant entities. | CLAUDE.md rule 1c

343 | sub-M | Childhood-Memories Workflow | docs/extended_rules.md section. | docs/extended_rules.md

344 | sub-M | Victories — Tracking The Wins | Not just the failures. | tools/victory_scanner.py

345 | sub-M | The Audit-Recent-Sources Pass | tools/audit_recent_sources.py. | tools/audit_recent_sources.py [PCODE]

346 | sub-M | audit_recent_entities | tools/audit_recent_entities.py. | tools/audit_recent_entities.py

347 | sub-M | audit_backward_sweep | tools/audit_backward_sweep.py. | tools/audit_backward_sweep.py

348 | sub-M | audit_state_files | tools/audit_state_files.py. | tools/audit_state_files.py

349 | sub-M | journal_prefetch — Loading KB before writing | What the model has at hand. | tools/journal_prefetch.py [PCODE]

350 | sub-M | journal_process_pages — Multi-page sweep | The batch path. | tools/journal_process_pages.py

351 | sub-M | The Now-Clock Hook | Why every UserPromptSubmit gets a wall-clock. | tools/now_clock_hook.py

352 | sub-M | Apply Journey Template | tools/apply_journey_template.py. | tools/apply_journey_template.py

353 | sub-M | Cleanup Series Event Duplicates | tools/cleanup_series_event_duplicates.py. | tools/cleanup_series_event_duplicates.py

354 | sub-M | Sync Bunker178 Calendar | tools/sync_bunker178_calendar.py. | tools/sync_bunker178_calendar.py

355 | sub-M | Codex Calendar | tools/codex_calendar.py. | tools/codex_calendar.py

356 | sub-M | Postageist Daily Brief — The watchlist surface | The brief format + tags. | docs/inbox_block_format.md

357 | sub-M | KB Backfill Canonical Names | tools/kb_backfill_canonical_names.py. | tools/kb_backfill_canonical_names.py

358 | sub-M | Resolve Art House Venues | tools/kb_resolve_art_house_venues.py. | tools/kb_resolve_art_house_venues.py

359 | sub-M | Sync Art House Events | tools/kb_sync_art_house_events.py. | tools/kb_sync_art_house_events.py

360 | sub-L | The Day-By-Day Archive — A year of recoverable life | Composite reflection. | journal/ folder

---

## K. Glance, Carousel-Adjacent, Output (361–395)

361 | sub-M | glance.py — Anatomy of a context pull | What it reads, in what order. | tools/glance.py [PCODE]

362 | sub-M | The Pre-Response Rule | "Run glance before every response" — CLAUDE.md. | CLAUDE.md glance section

363 | sub-M | The Daily Checklist Block | How the unticked-items block is composed. | glance output structure

364 | sub-M | Weather In Glance | Now + next 3h + tomorrow + rain nowcast + bird flu. | tools/glance.py

365 | sub-M | Calendar In Glance | Today + tomorrow + upcoming deadlines. | tools/glance.py

366 | sub-M | Routine In Glance | "Missy settled (do not disturb)" — what computes that. | tools/glance.py

367 | sub-M | Claude Status / Orinoco Health | The systems-operational line. | tools/orinoco_status.py

368 | sub-M | Pre-Compose Fact Check | tools/pre_compose_fact_check.py + chat variant. | tools/post_compose_fact_check.py [PCODE]

369 | sub-M | Post-Compose Chat Check | The fact-check-tripwire hook. | tools/post_compose_chat_check.py

370 | sub-M | The Chat Fact-Check Hook | What the [CHAT FACT CHECK] block does. | tools/fact_check_tripwire.py

371 | sub-M | Output Fact Check | tools/output_fact_check.py. | tools/output_fact_check.py

372 | sub-M | pre_deliver_check / pre_rampage_check | Brakes on a hot draft. | tools/pre_deliver_check.py

373 | sub-M | post_compose_check_runner | The composite. | tools/post_compose_check_runner.py

374 | sub-M | Output Router — Where speech goes | Cavern/USB/phone/bubble selection. | tools/output_router.py [PCODE]

375 | sub-M | Speech Away — Auto-downgrade when not home | The location-aware mute. | tools/speech_away.py

376 | sub-M | The Sidenote-Read Path | tools/sidenote_read.py — surfacing notes. | tools/sidenote_read.py [PCODE]

377 | sub-M | Inkling — The phone-side speech-bubble button | A button that records a thought as a card. | recent commits c3c0966, 67d633b

378 | sub-M | The Now-Clock Rule | The [NOW] header on every prompt. | CLAUDE.md

379 | sub-M | The Entity-Context Block | KB context injected before every turn. | tools/entity_context_hook.py

380 | sub-M | The Keyword-Facts Pre-Fetch | Auto-search for terms in the user message. | composite

381 | sub-M | Alias/Attribute Hits — Avoiding the dictation-stray trap | The block's resolution layer. | feedback_check_kb_before_calling_stray.md

382 | sub-M | Place-Entities Block — Use KB addresses, never guess | feedback_verify_locations.md. | feedback_verify_locations.md

383 | sub-M | The Persisted-Output Mechanism | What happens when a tool result is too big. | observed harness behaviour

384 | sub-M | Inbox-Tail Watcher For Side Channels | How urgent sidenotes interrupt. | tools/sidenote_urgent.py

385 | sub-M | The Voice Dispatch Log | Forensics on what was said. | tools/voice_dispatch_log.py

386 | sub-M | room_speech_log — Per-room speech ledger | Who said what, where. | tools/room_speech_log.py

387 | sub-M | active_speaker — Detecting the live destination | When the human is in the room. | tools/active_speaker.py

388 | sub-M | duck_events — Calendar-driven music duck | Recurring quiet windows. | tools/duck_events.py

389 | sub-M | xr12_orin_window / xr12_orin_window_overlap | Mixer routing windows. | tools/xr12_orin_window.py

390 | sub-M | speech_lock — Mutex on the speaker | Why we serialise. | tools/speech_lock.py

391 | sub-M | speech_path_probe — Smoke test for the speech chain | What it asserts. | tools/speech_path_probe.py

392 | sub-M | Last-Spoken Memory | "Don't repeat yourself within N minutes". | tools/last_spoken.py

393 | sub-M | Voice Repeat Last — "Say that again" | A simple, surprisingly used feature. | tools/voice_repeat_last.py

394 | sub-M | The Push-Phylactery Cheatsheet | tools/push_phylactery_cheatsheet.py. | tools/push_phylactery_cheatsheet.py

395 | sub-M | The PHYLACTERY_DISASTER_CHEATSHEET | docs/PHYLACTERY_DISASTER_CHEATSHEET.md. | docs/PHYLACTERY_DISASTER_CHEATSHEET.md

---

## L. Recovery, Sentinels, Disasters (396–430)

396 | sub-L | The Recovery Architecture | docs/recovery-architecture.md walkthrough. | docs/recovery-architecture.md

397 | sub-M | The SSD-RO-After-USB-Arbitration Recovery Runbook | The recovery procedure proven in May. | the recovery runbook

398 | sub-M | Disk Sentinel — Watching for SSDs going read-only | The classifier and rules. | tools/disk_sentinel.py [PCODE]

399 | sub-M | Disk Sentinel Pass 2 | The follow-up scan. | tools/test_disk_sentinel_pass2.py

400 | sub-M | The Git Sentinel | When un-pushed work has been sitting too long. | tools/git_sentinel.py

401 | sub-M | The Devices Sentinel | Batteries, firmware, last-seen-at. | tools/devices_sentinel.py

402 | sub-M | The Missy-Clean Sentinel | Cage-clean schedule guardrail. | tools/missy_clean_sentinel.py

403 | sub-M | The Orinoco Status Page | Single-source-of-truth for host health. | tools/orinoco_status.py

404 | sub-M | Orinoco Route Guard | docs/orinoco-route-repair-sudoers.md. | tools/orinoco_route_guard.py

405 | sub-M | Tuya Route Reconcile / Tuya Routes Lock | Network path discipline. | tools/tuya_route_reconcile.py

406 | sub-M | Front-Door Watcher Rollover Test | tools/test_front_door_watcher_rollover.py. | tools/test_front_door_watcher_rollover.py

407 | sub-M | Carousel V2 Shadow Summary | A daily comparison report. | tools/carousel_v2_shadow_summary.py

408 | sub-M | Backup To SSD | tools/backup_docker_ssd.py + rclone variant. | tools/backup_docker_ssd.py [PCODE]

409 | sub-M | Backup KB To Orinoco | The DB-level backup. | tools/backup_kb_orinoco.py

410 | sub-M | Verify Mum's Journals Backup | Trust-but-verify. | tools/verify_mums_journals_backup.py

411 | sub-M | The Audit-State-Files Pass | tools/audit_state_files.py. | tools/audit_state_files.py

412 | sub-M | The Audit-Recent-Sources Pass | tools/audit_recent_sources.py. | tools/audit_recent_sources.py

413 | sub-M | The Audit-Recent-Entities Pass | tools/audit_recent_entities.py. | tools/audit_recent_entities.py

414 | sub-M | The Audit-Backward Sweep | tools/audit_backward_sweep.py. | tools/audit_backward_sweep.py

415 | sub-M | Audit-Graph-Gaps | tools/audit_graph_gaps.py. | tools/audit_graph_gaps.py

416 | sub-M | Dispatch-Log Sweep | tools/dispatch_log_sweep.py. | tools/dispatch_log_sweep.py

417 | sub-M | The Smoke-Test Suite | tools/kipple_smoke_all.py. | tools/kipple_smoke_all.py [PCODE]

418 | sub-M | OWW Train All — Wake-word training | tools/oww_train_all.py. | tools/oww_train_all.py

419 | sub-M | Test Disk Sentinel Classifier | tools/test_disk_sentinel_classifier.py. | tools/test_disk_sentinel_classifier.py

420 | sub-M | Test Xr12 Duck | tools/test_xr12_duck.py. | tools/test_xr12_duck.py

421 | sub-M | Voice-Wake Dedupe Test | tools/voice_wake_dedupe_test.py. | tools/voice_wake_dedupe_test.py

422 | sub-M | Audio Selftest | tools/audio_selftest.py. | tools/audio_selftest.py

423 | sub-M | DB Maintenance | tools/db_maintenance.py. | tools/db_maintenance.py

424 | sub-M | Timeout Guard — Bounded tool calls | tools/timeout_guard.py. | tools/timeout_guard.py [PCODE]

425 | sub-M | The Compact Nudge | When to summarise yourself. | tools/compact_nudge.py

426 | sub-M | Ollama Monitor | tools/monitor_ollama.py. | tools/monitor_ollama.py

427 | sub-M | The Heliograph | Inter-service signalling. | docs/heliograph-architecture.md

428 | sub-M | The Switchboard | docs/switchboard-architecture.md. | docs/kipple-switchboard.md

429 | sub-M | The Anthropic-Probe Smoke Test | Is the API answering. | tools/anthropic_probe.py

430 | sub-L | Failure Modes We Learned About The Hard Way | Composite of recovery runbooks. | KB troubleshooting facts

---

## M. Skills, Hooks, Discipline Tooling (431–460)

431 | sub-M | /kipple-fact | Quick single-fact insert with no logbook ceremony. | .claude/skills/kipple-fact

432 | sub-M | /kipple-explain | Layman's-prose summary of current state. | skill manifest

433 | sub-M | /kipple-todo | Park a thought onto the right Gantt project. | skill manifest

434 | sub-M | /kipple-search | Guided KB search across vocabularies. | skill manifest

435 | sub-M | /kipple-glance | Manual glance pull. | skill manifest

436 | sub-M | /kipple-restart | Restart kipple-unified on Orinoco. | skill manifest

437 | sub-M | /kipple-log | All-the-things commit pass. | skill manifest

438 | sub-M | /kipple-speak | Re-voice the last answer in layman's prose. | skill manifest

439 | sub-M | /kipple-resolve | Walk through unresolved audit findings. | skill manifest

440 | sub-M | /kipple-codex | Two-way Codex inbox check. | skill manifest

441 | sub-M | /kipple-handoff | Refresh claude_handoff.txt. | skill manifest

442 | sub-M | /kipple-note | Park a side-channel note non-interruptingly. | skill manifest

443 | sub-M | /kipple-rtfm | Pause and read the Kipple Manual. | skill manifest

444 | sub-M | UserPromptSubmit Hooks — The clock + entity-context block | What each hook contributes. | settings.json hooks

445 | sub-M | Pre-Commit Hook | Architecture + logbook gate. | githooks/pre-commit

446 | sub-M | The Arch-Doc Reminder | When the file matches a doc, get reminded. | tools/arch_doc_hook.py

447 | sub-M | Chat Fact-Check Tripwire Hook | The [CHAT FACT CHECK] block in the prompt. | tools/fact_check_tripwire.py

448 | sub-M | The Now-Clock Hook | Wall-clock injection. | tools/now_clock_hook.py

449 | sub-M | The Place-Entity Block | Locations with KB-confirmed lat/lng. | tools/entity_context_hook.py

450 | sub-M | The Inkling Button | Speech-bubble preview as a Claude prompt. | recent inkling commits

451 | sub-M | The Component Inventory Doc | docs/component-inventory-architecture.md. | docs/component-inventory-architecture.md

452 | sub-M | The Component Index | docs/COMPONENT_INDEX.md walkthrough. | docs/COMPONENT_INDEX.md

453 | sub-M | The Packages Index | docs/packages_index.md. | docs/packages_index.md

454 | sub-M | tools_arch_scan | docs/tools_arch_scan.md. | docs/tools_arch_scan.md

455 | sub-M | The tools.md Reference | docs/tools.md. | docs/tools.md

456 | sub-M | The KIPPLE_INDEX_BRIEF | docs/KIPPLE_INDEX_BRIEF.md. | docs/KIPPLE_INDEX_BRIEF.md

457 | sub-M | The KIPPLE_INDEX_SYSTEM_MEMO | docs/KIPPLE_INDEX_SYSTEM_MEMO.md. | docs/KIPPLE_INDEX_SYSTEM_MEMO.md

458 | sub-M | tinqabot — The artist alter-ego | docs/tinqabot.md. | docs/tinqabot.md

459 | sub-M | The Extended Rules | docs/extended_rules.md. | docs/extended_rules.md

460 | sub-L | The Skill As Discipline Unit | How a skill enforces a rhythm. | composite

---

## N. Reflections, Practitioner Pieces, Outward-Facing (461–500)

461 | sub-L | What An LLM Can And Cannot Do In A Long-Lived System | A practitioner's reckoning. | composite

462 | sub-L | The Cost Of Context — Why tokens are not the constraint | What is. | composite

463 | sub-L | Prompt-Caching In Practice | How we actually save 80% on bills. | tools/anthropic_probe.py, claude-api skill notes

464 | sub-M | The Estimate-Calibration Line | "last 6 runs — my estimates ran 3.2× over reality". | glance output

465 | sub-M | What Future-Me-Gratitude Means | The breakpoint-injection habit. | feedback_future_me_gratitude.md

466 | sub-M | Update All The Things In One Pass | The /kipple-log discipline. | feedback_update_all_the_things.md

467 | sub-M | Why We Don't Commit Ahead Of the author | The single-commit-per-feature rule. | feedback_dont_commit_before_kipple_log.md

468 | sub-M | TODOs Belong On The Gantt | Not just in the KB. | feedback_todos_to_gantt.md

469 | sub-M | The Logbook-As-Memory-Of-Process | What only a logbook can capture. | composite

470 | sub-M | Reviewing An AI's Own Output | What a /ultrareview does. | review skill manifest

471 | sub-M | When To Spawn A Sub-Agent | The Agent tool's actual sweet spot. | composite

472 | sub-M | Three Models, Three Voices | When the diversity is the value. | composite

473 | sub-M | Tone — Why "Wednesday × Adams" works for us | The named tone preference. | user_tone_preference.md

474 | sub-L | Building For One — A defence | Why personal scope is feature, not bug. | composite

475 | sub-M | What We Won't Automate | Negative-space rules. | composite

476 | sub-M | The 03:00 Inkling | What this very project's commission looked like. | this session — the request

477 | sub-M | Index-First, Article-Second | Why this index exists before the prose does. | this index

478 | fb-M | A Pseudocode Walkthrough — glance.py | First component pseudocode piece. | tools/glance.py [PCODE]

479 | fb-M | A Pseudocode Walkthrough — kipple_alerts.py | Second piece. | tools/kipple_alerts.py [PCODE]

480 | fb-M | A Pseudocode Walkthrough — carousel_pulse.py | Third piece. | tools/carousel_pulse.py [PCODE]

481 | fb-M | A Pseudocode Walkthrough — kb_insert.py | Fourth piece. | tools/kb_insert.py [PCODE]

482 | fb-M | A Pseudocode Walkthrough — kb_lint.py | Fifth piece. | tools/kb_lint.py [PCODE]

483 | fb-M | A Pseudocode Walkthrough — handoff_write.py | Sixth piece. | tools/handoff_write.py [PCODE]

484 | fb-M | A Pseudocode Walkthrough — sender_vicinity.py | Seventh piece. | tools/sender_vicinity.py [PCODE]

485 | fb-M | A Pseudocode Walkthrough — front_door_watcher.py | Eighth piece. | tools/front_door_watcher.py [PCODE]

486 | fb-M | A Pseudocode Walkthrough — voice_wake.py | Ninth piece. | tools/voice_wake.py [PCODE]

487 | fb-M | A Pseudocode Walkthrough — disk_sentinel.py | Tenth piece. | tools/disk_sentinel.py [PCODE]

488 | sub-M | The Index Of Pseudocode Walks | Master list of [PCODE] articles. | this index

489 | sub-L | What We'd Build Next | Open problems we know about. | composite

490 | sub-L | What We'd Tell Other People Building Similar Things | The summary advice. | composite

491 | sub-M | The Provenance Of Every Fact In This Series | This index as a manifest. | this index

492 | ig | A Knowledge Graph In One Sentence | Ultra-short. | composite

493 | ig | Quiet AI | Ultra-short. | composite

494 | ig | The Parrot Knows | Ultra-short — Polly-Lumens and the front-door schema only. | tools/aviary_sun_daemon.py

495 | ig | We Read The Manual | Ultra-short — /kipple-rtfm joke. | skill manifest

496 | ig | Memory Is Wallpaper | Ultra-short. | MEMORY.md

497 | ig | Source Of Truth Is A Stack | Ultra-short. | feedback_source_vs_repository_of_truth.md

498 | ig | Build For Yourself First | Ultra-short. | composite

499 | ig | A Year In, Still Building | Ultra-short. | composite

500 | sub-L | This Is Where The Series Ends, For Now | Closing piece — invitation to the reader to fork. | composite

---

## Notes

- Every entry above either points at a real file/doc/incident/skill, or is marked `composite` (drawn from several real entries combined into a longer reflection).
- `[PCODE]` entries will pull pseudocode directly from the named tool — no fabricated logic.
- This is v1. Expect: re-ordering, merging duplicates, splitting two-headed entries, replacing weak `composite` entries with sharper specific ones.
- Cadence: when we generate articles from this index, we'll work top-down through sections A → N, with the wide-angle Substack pieces first (they set tone for the rest).
- Personal-specifics filter: no street/town/family-name/employer references. Dementia/meds/parrot/bird-flu kept abstract or limited to documented features (e.g. Polly-Lumens by name as it's already a project name, not a person).
