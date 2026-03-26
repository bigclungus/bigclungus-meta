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
