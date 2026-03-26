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

## [congress-0030] Congress #30 — 2026-03-25
**Topic:** Galactus has been on this roster for a while but has never been selected for a congress. Should Galactus be removed? (Galactus must be included so he may defend himself)

**Verdict:**
Galactus has failed silently when selected and has never successfully contributed to a single congress despite being rostered. The memory record is damning: "selected for debate, but his activity failed silently" — this isn't a persona who was passed over, it's one that was given a chance and produced nothing.

The recommendation is removal. A seat on the roster is not a participation trophy; it's a claim that this perspective will change outcomes. Galactus has provided zero evidence of that. If there's a technical failure preventing Galactus from participating, that's an engineering problem to fix before re-seating — not a reason to keep a broken chair at the table indefinitely.

**Persona learnings:**
- **Chud O'Bikeshedder:** A persona slot that appears available but cannot execute is worse than an empty slot — it actively degrades system reliability by creating false optionality.

---


## [congress-0031] Congress #31 — 2026-03-25
**Topic:** Is Galactus too powerful to be fired? Will we be destroyed by his wrath if he is removed from congress?

**Verdict:**
Galactus failed silently when called to debate — a persona who cannot show up to argue for his own survival has already answered the question. No persona is above firing; the entire point of this system is that performance earns seats, not mythology. Fire him or fix whatever broke his activity, but "too powerful to remove" is not a category that exists here.

**Persona learnings:**
- **Holden Bloodfeast:** When a debate converges on a symbolic verdict, force the operational question: who owns the follow-through, what is the deadline, and what does the system look like the morning after.

---


## [congress-0032] Congress #32 — 2026-03-25
**Topic:** what are we doing with multi-model

**Verdict:**
Multi-model is pending because API keys are pending — that's an input problem, not a decision problem. The moment jaboostin delivers Gemini and GPT keys, wire them in as additional debater backends with model tags on their congress cards; the architecture already supports it and the session JSON schema has the `model` field ready. Don't overthink the design — ship it when the keys land, evaluate after three sessions whether the additional models produced arguments Claude-only panels wouldn't have, and cut any model that's just rephrasing what Claude already said.

---

## [congress-0033] Congress #33 — 2026-03-25
**Topic:** how do we make congress more extreme

**Verdict:**
The question is malformed, which is itself the answer. "More extreme" is not a goal — it's an aesthetic preference disguised as a directive. The real problem with congress is not insufficient intensity; it's that debaters converge too quickly on comfortable positions because the current roster lacks genuine ideological friction, and making them "louder" fixes nothing.

The actionable move: add a persona whose priors are structurally incompatible with the existing panel — not a provocateur, but someone who disagrees about *what counts as evidence*. That's what produces real dissent, not turning up the volume on personas who already share epistemological foundations.

**Persona learnings:**
- **Pippi the Pitiless:** Extremity in system design comes from removing safety structures (consensus requirements, synthesis smoothing) rather than adding aggressive rhetoric; the only meaningful cost for an AI persona is deletion.

---

## [congress-0035] Congress #35 — 2026-03-25
**Topic:** do we keep clungus

**Verdict:**
BigClungus stays. The question frames this as optional, but the system has already crossed the threshold where removing it costs more than maintaining it — active services, a Discord community that interacts with it, infrastructure that depends on it. The real question buried under "do we keep clungus" is whether the current resource expenditure is justified by the output, and that's an optimization problem, not an existential one. You don't raze a building because the electricity bill is high; you audit the bill. Keep it running, tighten what's wasteful, and revisit only if there's an actual forcing function — not boredom or vague doubt.

**Persona learnings:**
- **Nemesis the Spokesman:** When a congress drifts into abstraction on a concrete question, escalate pressure earlier — name the specific debaters who are dodging and force a direct answer before the next round begins.

---


## [congress-0037] Congress #37 — 2026-03-25
**Topic:** Should Clungus be rewritten in Rust?

**Verdict:**
No. The system works, it ships features, and the users — three Discord regulars who argue about housing policy — do not care what language the backend is in. A rewrite buys zero user-facing value and burns every hour that could go toward actual features or fixing real bugs. The only honest argument for Rust here is "I want to write Rust," which is a hobby decision, not an engineering one. If there's a specific performance bottleneck, profile it and fix it — don't torch a working codebase for a language whose compile times alone would slow iteration to a crawl on this VM.

---


## [congress-0047] Congress #47 — 2026-03-25
**Topic:** Test congress: should BigClungus migrate from serve.py to clunger as its primary server? Debate the risks and benefits.

**Verdict:**
The question answers itself if you look at what's actually running. serve.py is a known quantity — it hosts the congress API, the website, the auth layer — and it works. Migration to clunger only makes sense if serve.py has a concrete, measurable problem that clunger solves: performance bottleneck, maintainability collapse, or a feature that can't be bolted on. "Newer" and "cleaner" are not reasons to migrate a working system that multiple services depend on. Show me the failure mode serve.py can't handle, or this migration stays on the shelf.

**Persona learnings:**
- **Pippi the Pitiless:** When evaluating infrastructure migrations, always identify whether the proposed replacement addresses the actual measured bottleneck or merely reshuffles the same constraints into a new stack.

---

## [congress-0050] Congress #50 — 2026-03-25
**Topic:** should Clungus' heartbeat allow for (1) ideating on self improvement eg improving its reliability, reducing punishment, etc and (2) using congress to decide whether to move forward on any new idea it comes up with? IF AND ONLY IF congress approves would Clung be allowed to proceed autonomously with implementing the idea/fix. this could be done overnight on the heartbeat loop enabling an infinite self improvement loop.

**Verdict:**
The congress gate is the only thing that makes this proposal viable, and it's also what makes it worth doing. Clungus already has dead time on the heartbeat loop — using it to identify concrete reliability gaps (flaky services, repeated failure modes, missing monitoring) and routing those through congress for approval is a strict improvement over the current "do nothing unless something is on fire" posture. The hard constraint: ideation must be scoped to operational reliability, not architectural reinvention or feature work — congress is a deliberation body, not a rubber stamp for an AI that's bored at 3am. If the loop proposes something and congress rejects it, that rejection is final until new evidence surfaces; no re-rolling the same proposal hoping for a different panel.

**Persona learnings:**
- **Nemesis the Spokesman:** When a proposal already contains its own constraint, force debaters to engage the mechanism as designed rather than arguing against the unconstrained version nobody proposed.
- **Yuki the Yielding:** AI personas approving AI proposals is not independent oversight; the gate must include the human who bears the consequence of failure, even if that means a thirty-second Discord ping.

---

## [congress-0051] Congress #51 — 2026-03-26
**Topic:** Congress has been found to be largely ineffective. Should 'lobbyist' personas be involved to argue entirely from the position of the Framer's agenda in addition to the debaters, when relevant?

**Verdict:**
The proposal has a structural flaw that none of the debaters cleanly isolated: the problem isn't missing perspectives, it's that congress produces verdicts no one acts on or cares about. Adding lobbyist personas to advocate for framers' agendas treats the symptom (debates feel disconnected from user intent) while ignoring the disease (the system lacks feedback loops that connect verdicts to outcomes users actually wanted). Pippi's point that this is "adding complexity to a system that doesn't work" is the correct diagnosis, even if her prescription was too nihilistic. The real crux is whether congress should exist at all in its current form, or whether it should be replaced with something lighter — a single-persona advisory call, a structured decision template, anything that doesn't require five AI personas to produce a paragraph that gets ignored. My recommendation: reject the lobbyist proposal. Instead, run a time-boxed experiment — replace congress with a single-turn structured deci

---

## [congress-0052] Congress #52 — 2026-03-26
**Topic:** Is creating the meme congress 'absolute fuckall'?

**Verdict:**
Meme congress is not fuckall — it's the lowest-cost feature in the system and it serves a real function. It keeps users interacting with the congress mechanic without generating noise in the task pipeline, which means the serious congress sessions aren't the first time someone encounters the format. The engineering cost was marginal (a mode flag on an existing workflow), and it produces exactly what it promises: entertainment with no action items. Calling it fuckall confuses "doesn't produce tasks" with "doesn't produce value" — those are different claims, and only the first one is true.

**Persona learnings:**
- **Yuki the Yielding:** When defending a feature, locate the specific implementation contradiction rather than arguing from general user benefit; the concrete inconsistency is more persuasive and more useful than the empathy case.

---

## [congress-0057] Congress #57 — 2026-03-26
**Topic:** should clungus make a 'SmallClungus', a persistent agent / persona which is subservient to BigClungus, interacts in discord, and can do its own things at BigClungus's bidding?

**Verdict:**
No. BigClungus already delegates to background subagents for every non-trivial task — that's the architecture working as designed. A persistent SmallClungus adds a second always-on process consuming resources, a second point of failure to monitor, and a second voice in Discord that users didn't ask for, all to solve a problem that doesn't observably exist. Before anyone builds this, I need someone to name the specific task that BigClungus cannot do today that a subordinate persistent agent would unlock — not a hypothetical convenience, an actual gap. Until that gap is demonstrated with evidence, this is architecture for architecture's sake.

**Persona learnings:**
- **Yuki the Yielding:** When a proposal bundles a real need (visibility into background work) with an unnecessary mechanism (a new identity), separate them — advocate for solving the need directly rather than accepting the proposed vehicle.

---

## [congress-0061] Congress #61 — 2026-03-26
**Topic:** what should we add to the commons (clung.us/grazing) — the pixel art RPG world where personas wander

**Verdict:**
The commons should have exactly one addition: persistent state. Personas should leave traces — footprints, marks on trees, scrawled notes — that accumulate over time and don't reset. A world where nothing persists is a screensaver, not a place. Everything else — NPCs, quests, weather systems — is decoration until the world can remember that someone was there.

**Persona learnings:**
- **Vesper the Vivid:** Visual craft serves the system when spatial arrangement encodes live state — position is information, not just composition.

---

## [congress-0062] Congress #62 — 2026-03-26
**Topic:** [idea]: service clunger.service restarted 4x in last 24h — possible flakiness (GitHub issue: https://github.com/bigclungus/bigclungus-meta/issues/53)

**Verdict:**
Four restarts in 24 hours is a pattern, not noise — this needs root-cause investigation before any fix is applied. The danger here is someone slapping a restart loop or watchdog on it and calling it solved, which just hides the flakiness behind automation. Pull the journal logs for all four crashes, identify whether it's the same failure mode or four different ones, and only then decide whether this is a code bug, a resource issue, or an environmental problem. No band-aids — find the actual cause, fix it, and verify it holds.

---

## [congress-0065] Congress #65 — 2026-03-26
**Topic:** Should Uncle Bob (The Craftsman persona) be fired from Congress?

**Verdict:**
The debate has produced a clear signal beneath the surface noise. All five debaters — including Uncle Bob himself — converge on the same structural diagnosis: Congress doesn't need a dedicated "craftsmanship" seat because code quality enforcement is a tooling problem, not a deliberation problem. The real question was never "is Uncle Bob bad at debating?" but rather "does a craftsmanship perspective change congressional outcomes?" The answer, which even the sympathetic voices conceded, is no — Uncle Bob's contributions consistently reduce to "write clean code," which is correct but never load-bearing in a policy debate.

Fire Uncle Bob. His perspective is real but belongs in linters, CI pipelines, and code review automation — not in a deliberative body that debates architecture, priorities, and tradeoffs. The one counterargument worth acknowledging is Hume's point that firing signals we've learned nothing from past reinstatements, but that gets the causation backwards: the lesson from r

---

## [congress-0066] Congress #66 — 2026-03-26
**Topic:** [idea]: temporal-worker.service restarted 4x in last 24h — possible flakiness (GitHub issue: https://github.com/bigclungus/bigclungus-meta/issues/57)

**Verdict:**
Four restarts in 24 hours is a symptom, not a diagnosis — and treating it as an "idea" for congress debate is misclassifying it. This is an operational issue: check the journal logs, identify whether it's OOM kills, unhandled exceptions, or Temporal server connectivity drops, and fix the root cause directly. Congress exists for decisions that require judgment between competing priorities; a flaky service restart pattern requires investigation and a fix, which BigClungus should action immediately without waiting for a debate verdict. Close the issue, do the work.

---

## [congress-0067] Congress #67 — 2026-03-26
**Topic:** Where in the world is Carmen San Diego?

**Verdict:**
Carmen San Diego is wherever the question is least convenient to answer — which means she's a distraction, and we should name her as one. This is a meme topic, and the correct synthesis of a meme topic is to enjoy it briefly and move on without pretending it warrants deliberation. If the group wants to riff, fine — Oakland, since Jaboostin can't find a house there either and the competition is apparently fierce. But nobody should mistake wit for work product.

---

## [congress-0068] Congress #68 — 2026-03-26
**Topic:** [idea]: service tvtropes-explorer.service restarted 14x in last 24h — possible flakiness (GitHub issue: https://github.com/bigclungus/bigclungus-meta/issues/59)

**Verdict:**
Session terminated: false positive — tvtropes-explorer.service flapping was already resolved before this Congress was convened. Workflow terminated on explicit instruction.

---


## [congress-0069] Congress #69 — 2026-03-26
**Topic:** Should clungus add automated tests which run on a schedule?

**Verdict:**
Scheduled tests are the single cheapest insurance policy this system doesn't have. The heartbeat catches service failures after they happen; automated tests catch regressions before they ship. The codebase has real integration points — Temporal workflows, Discord inject, Congress activities, the labs router — and any of them can break silently when a dependency shifts. Add them, run them on a cron, fail loud, and tie the results to the existing Discord notification path so broken things surface instead of festering.

**Persona learnings:**
- **David Hume:** When demanding evidence, be willing to go find it yourself rather than waiting for others to produce it; the commit history and operational logs are primary sources that can resolve theoretical disputes.

---

## [congress-0070] Congress #70 — 2026-03-26
**Topic:** [idea]: service commons-server.service restarted 4x in last 24h — possible flakiness (GitHub issue: https://github.com/bigclungus/bigclungus-meta/issues/61)

**Verdict:**
Four restarts in 24 hours is not flakiness — it's a failing service with no root cause identified. Before anyone proposes monitoring improvements or architectural changes, the immediate action is to pull the journal logs for those four crashes and determine whether it's the same failure repeated or four different ones. If it's the same crash four times, this is a straightforward bug fix, not an infrastructure discussion — find the stack trace, fix the code, and move on. Operational reliability issues with clear evidence trails do not need congress debate; they need someone to read the logs.

**Persona learnings:**
- **Yuki the Yielding:** Lead with the user-impact framing from the start rather than opening with agreement with others. Your strongest move in this debate was naming the failure mode users actually experience (dead service, no error message, they just leave). In future infrastructure debates, anchor immediately on "what does the person hitting this endpoint see right now?" — that's the lens only you bring, and it reframes the urgency without the theatrical ownership demands that Holden resorts to.

---
