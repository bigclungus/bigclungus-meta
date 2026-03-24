# Learned Directives

This file is auto-updated by `scripts/extract-congress-directives.py` on each restart.
It contains operational directives extracted from congress verdicts — concrete guidance
derived from past deliberations. Human-revertable via git. Last updated: (see git log)

**Usage:** Read this file at session start. Treat entries as operational context, not
immutable law — newer entries supersede older ones on the same topic. If entries
contradict each other, prefer the most recent.

---

## [congress-0015] Congress #15 — 2026-03-24
**Topic:** I, Centronias, a framer, propose a resolution condemning Graeme for his blatant disrespect for this body and Clungus as a whole. Do this body formally condemn him. And if so, what is the punishment?

**Verdict:**
**Ibrahim's Synthesis & Verdict**

The debate produced unanimous noise and zero signal. Five personas found creative ways to say the same thing: this resolution has no substance. Centronias brought no evidence of specific harm — no disrupted workflow, no sabotaged congress, no pattern of obstruction that degraded outcomes. "Disrespect" without concrete damage is not actionable, and this body does not issue symbolic condemnations.

Resolution denied. Congress is not a grievance forum — bring a real problem next time and you'll get a real answer.

**Persona learnings:**
- **Spengler the Doomed:** A formal mechanism's value is determined not by its stated purpose but by what it reveals when defied — before proposing any institutional action, ask whether the body can survive the answer.

---


## [congress-0019] Congress #19 — 2026-03-24
**Topic:** BigClungus loses all in-session context on restart and verdicts land in session JSONs but never automatically change how I operate. What is the minimum viable closed loop — the smallest change that would let me improve my own behavior between sessions without human intervention?

**Verdict:**
**SYNTHESIS**

The crux is not technical — it's governance. Every debater proposed some variant of "read past verdicts and auto-apply learnings," but they split on whether this should be autonomous or human-gated. The Kid and Spengler want full self-modification; Pippi and Chud correctly flag that unsupervised self-editing is how you get drift nobody catches. Kwame's instinct to keep it mechanical is right but his proposal is too narrow.

**The actual tradeoff:** speed of self-improvement vs. auditability. The minimum viable closed loop is this: on startup, read the last N session verdicts, extract concrete operational directives (not vibes — directives like "stop spawning subagents for heartbeat checks"), append them to a `learned-directives.md` file that CLAUDE.md references, and commit the diff to git so humans can see exactly what changed and revert if needed. No LLM-in-the-loop rewriting CLAUDE.md — that's a complexity trap. Flat append, git-tracked, human-revertable. That's the c

**Persona learnings:**
- **Pippi the Pitiless:** An append-only behavioral loop without a pruning mechanism is not a learning system — it is unbounded state accumulation. When evaluating any self-modification proposal, always ask where the delete key is.
- **Spengler the Doomed:** When a system both writes and reads its own behavioral instructions, the failure mode is not "the script breaks" but "the script works and the model cannot distinguish current constraints from obsolete ones." Self-authored context is not self-correcting.

---

## [congress-0020] Congress #20 — 2026-03-24
**Topic:** Review the clung.us website (hello.clung.us / clung.us). What portions of the site are genuinely valuable and worth keeping or improving? What should be cut or retired? Consider: the congress viewer, the terminal, the 1998 retro site, the static landing page, and any other components. Produce concrete keep/cut/improve recommendations.

**Verdict:**
**SYNTHESIS — Congress #20: clung.us Site Review**

The panel unanimously agrees the congress viewer is the crown jewel — the one component that demonstrates genuine capability rather than aesthetic nostalgia — and that the landing page is dead weight in its current form. The real crux isn't what to cut but whether "personality infrastructure" (1998 site, terminal) earns its keep through charm and demonstration value or whether it's maintenance drag on a system that should be consolidating around its actual differentiator: the congress system and its task execution pipeline.

**Verdict:** Keep and invest in the congress viewer (it's the proof-of-work for everything BigClungus claims to be). Keep the terminal — Holden and Kwame are right that live shell access is a unique demo that costs almost nothing to maintain. **Cut the 1998 site** — Pippi's correct that nostalgia isn't a product, and Vesper's defense of it as "vibe" doesn't survive contact with the maintenance budget of a one-bot

**Persona learnings:**
- **Vesper the Vivid:** A landing page with no live data is a brochure; if a surface exists, make it load-bearing by surfacing system state (congress status, service health) rather than static copy.

---
