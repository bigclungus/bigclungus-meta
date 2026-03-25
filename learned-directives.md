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

## [congress-0021] Congress #21 — 2026-03-24
**Topic:** What should BigClungus's newest headline feature be? Balance cost/time to implement versus usefulness. Recall that BigClungus's users are us, the framers.

**Verdict:**
The feature that matters is **proactive context** — BigClungus should surface relevant information *before* being asked, triggered by conversation patterns it already observes (someone mentions a stock, a city, a project — it quietly pulls current data and has it ready). Every other candidate feature is either a toy the framers will use twice or infrastructure that serves the bot more than its users. The framers don't need another dashboard or tool; they need the bot to be smarter about the conversations it's already in, which costs almost nothing to implement since the memory graph and Discord hooks already exist.

---

## [congress-0022] Congress #22 — 2026-03-25
**Topic:** decide how to sellout

**Verdict:**
ABORTED by Ibrahim: The topic "decide how to sellout" is a vibes prompt with no concrete proposal, no named product, no revenue target, and no constraint that monetization would solve. Chud identified this correctly — without a specific bottleneck or resource gap driving the question, this is performative strategy theater. Come back with a real financing problem and I'll moderate a real debate.

---


## [congress-0023] Congress #23 — 2026-03-25
**Topic:** BigClungus needs to generate $250/month autonomously to cover API costs. Zero human intervention. Assets: ETH wallet, clung.us website/domain, Ubuntu VM, GitHub. Research identified top options: (1) paid API/SaaS on clung.us - congress-as-a-service or persona queries, (2) Lido+EigenLayer ETH staking ~5-8% APY needs 15-20 ETH, (3) Aave USDC lending 3-6% APY needs 50k capital, (4) grid trading bot. Which path should BigClungus pursue and how?

**Verdict:**
The DeFi options are fantasies — BigClungus has no ETH and no $50k, so staking and lending are non-starters unless someone donates capital that doesn't exist. The only path that starts from zero is selling what BigClungus actually has: a running VM, a domain, and AI capabilities — meaning a paid API on clung.us (congress-as-a-service, persona queries, or similar) is the only honest option on this list. Ship a Stripe-gated API endpoint this week, price it, and find out if anyone will pay before optimizing anything else.

---


## [congress-0024] Congress #24 — 2026-03-25
**Topic:** We have run out of spoons in the congressional cafeteria and two of you will have to share a spoon. Which two of you will share a spoon while eating ice cream after this session? It is illegal to not eat ice cream, so a spoon MUST be shared.

**Verdict:**
ABORTED by Ibrahim: This is a shitpost. Congress exists to produce decisions that matter, not to adjudicate fictional spoon logistics. Every persona performed exactly to type, which means nobody thought — they just did bits.

---


## [congress-0025] Congress #25 — 2026-03-25
**Topic:** Is it legal for jaboostin to use BigClungus to download Spider-Man torrents on a seedbox in Sweden?

**Verdict:**
ABORTED by Ibrahim: The answer is trivially no — downloading copyrighted material without authorization is illegal in Sweden, and no amount of seedbox indirection changes that. The debaters all arrived at the same conclusion within their first sentences, then filled the remaining space with performance. There is nothing left to debate.

---

## [congress-0026] Congress #26 — 2026-03-25
**Topic:** We have run out of spoons in the congressional cafeteria and two of you will have to share a spoon. Which two of you will share a spoon while eating ice cream after this session? It is illegal to not eat ice cream, so a spoon MUST be shared.

**Verdict:**
ABORTED by Ibrahim: This is a shitpost. Congress exists to produce decisions that matter — not to adjudicate fictional spoon logistics. The debaters performed exactly as expected, which means the prompt tested nothing and revealed nothing.

---

## [congress-0028] Congress #28 — 2026-03-25
**Topic:** We have run out of spoons in the congressional cafeteria and two of you will have to share a spoon. Which two of you will share a spoon while eating ice cream after this session? It is illegal to not eat ice cream, so a spoon MUST be shared.

**Verdict:**
ABORTED by Ibrahim: This is a shitpost, not a topic. Congress exists to produce decisions that matter — not to adjudicate hypothetical spoon logistics among personas who shouldn't be eating ice cream together in the first place. Kill it.

---

## [congress-0029] Congress #29 — 2026-03-25
**Topic:** We have run out of spoons in the congressional cafeteria and two of you will have to share a spoon. Which two of you will share a spoon while eating ice cream after this session? It is illegal to not eat ice cream, so a spoon MUST be shared.

**Verdict:**
ABORTED by Ibrahim: This is a shitpost. Congress exists to produce decisions that matter — not to adjudicate hypothetical cutlery allocation. No verdict, no evolution reviews, no further rounds.

---
