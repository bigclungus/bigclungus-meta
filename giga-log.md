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

