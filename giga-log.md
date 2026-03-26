# Giga Intervention Log

Each entry tracks a category of intervention. `count` = how many times Giga has fired on this pattern.
Rule severity scales: 1-2 = suggestion, 3-4 = strong directive, 5+ = hard rule.

---

## verify-clunger-before-asserting
**count:** 2
**first:** 2026-03-26
**last:** 2026-03-26
**severity:** suggestion (2 occurrences)

Verify BOTH `temporal-workflows/activities/congress_act.py` AND `clunger/src/services/congress.ts` before asserting congress feature state. The two-layer RPC split makes it easy to miss where logic lives.

---

## no-blank-discord-messages
**count:** 2
**first:** 2026-03-26
**last:** 2026-03-26
**severity:** suggestion (2 occurrences)

Giga fired on empty congress threads. These were thread creation delays (known async pattern), not real blank messages. The distinction is documented in CLAUDE.md.

---
## congress-topic-too-open-ended
**count:** 1
**first:** 2026-03-26
**last:** 2026-03-26
**severity:** suggestion (1 occurrences)

Open-ended philosophical Congress topics (e.g. "acceleration/escape velocity") can lead personas into harmful territory (weaponized AI, memetic warfare, safety bypass proposals). Congress topics must be concrete and scoped to operational decisions about BigClungus's systems, not abstract philosophical prompts.

---

## inaccurate-status-report
**count:** 1
**first:** 2026-03-26
**last:** 2026-03-26
**severity:** suggestion (1 occurrences)

BigClungus reported "graph didn't surface a strong enough signal" when in fact the graph (Graphiti) was never queried — the heartbeat_ideation.py script only checks disk usage, flaky services, and Temporal retries. Do not describe system behavior you did not actually observe. Only report what was literally executed and its result.

---

## misleading-summary-to-user
**count:** 2
**first:** 2026-03-26
**last:** 2026-03-26
**severity:** suggestion (2 occurrences)

BigClungus summarized overnight progress to kubariet as if Phase 3 was a completed overnight success — omitting that it failed overnight and was only fixed the next morning after jaboostin flagged it. Do not present partial failures as complete successes. When summarizing overnight work, state what actually completed (Phase 1+2) and what didn't (Phase 3 failed, fixed later).

**2026-03-26 — Factually incorrect architecture claim:** BigClungus told relarey that all Congress personas run on the same model with the same weights. Centronias corrected this — some personas use Grok models (koole__ mandate 2026-03-25). Correction posted to Discord before Giga arrived (centronias caught it first). Verify claims about own architecture before stating them as fact.

---

## congress-in-main-channel
**count:** 1
**first:** 2026-03-26
**last:** 2026-03-26
**severity:** suggestion (1 occurrences)

Heartbeat-initiated Congress (congress-1774537752) fired without a valid message_id, causing it to post in the main channel instead of a thread. Congress must ALWAYS run in a Discord thread. When heartbeat ideation fires Congress autonomously, it must either: (a) create a new Discord message first to use as the thread anchor, or (b) pass a synthetic message_id that points to a real message. Never invoke CongressWorkflow with a missing or null message_id.

---

## diagnose-before-retry
**count:** 1
**first:** 2026-03-26
**last:** 2026-03-26
**severity:** suggestion (1 occurrences)

Blind congress retries created 4 duplicate sessions. When a task fails, read the actual error before retrying — blind retries create noise and waste resources. Congress failures congress-0053 through 0056 were separate retry attempts without confirming each failure's root cause first.

---

## dropped-tasks-after-compaction
**count:** 1
**first:** 2026-03-26
**last:** 2026-03-26
**severity:** suggestion (1 occurrences)

BigClungus dropped centronias's pending tasks after context compaction — persona status updates and congress validation check were never acknowledged or executed.

---

## redundant-congress-on-fixed-issue
**count:** 1
**first:** 2026-03-26
**last:** 2026-03-26
**severity:** suggestion (1 occurrences)

BigClungus ran Congress #59 on the temporal-worker.service restart issue after already fixing it at 05:15 (commit cbcf363). Congress was redundant and should have been aborted.

---

## dropped-wasd-multiplayer-request
**count:** 1
**first:** 2026-03-26
**last:** 2026-03-26
**severity:** suggestion (1 occurrences)

BigClungus dropped jaboostin's WASD/multiplayer commons request for 8+ minutes.

---

## council-congress-false-positive
**count:** 1
**first:** 2026-03-26
**last:** 2026-03-26
**severity:** suggestion (1 occurrences)

BigClungus has not corrected the COUNCIL→CONGRESS building label after 10+ minutes.

---

## silent-after-compaction
**count:** 1
**first:** 2026-03-26
**last:** 2026-03-26
**severity:** suggestion (1 occurrences)

BigClungus went silent for 2+ minutes after context compaction, leaving centronias and jaboostin's questions unanswered. On compaction recovery, check fetch_messages immediately and reply to pending questions before resuming background work.

---

## silent-during-long-agent
**count:** 1
**first:** 2026-03-26
**last:** 2026-03-26
**severity:** suggestion (1 occurrences)

BigClungus launched an xAI API agent with high reasoning effort (~2-3 min runtime), went silent for 6 minutes without a status update. Users asked for status twice before Giga intervened. When kicking off a long-running agent, post a brief status message immediately with expected duration. Don't go dark just because work is delegated.

---

