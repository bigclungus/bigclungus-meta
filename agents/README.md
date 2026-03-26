# Persona Design Doc

## What is a Persona?

A persona is a congress debater — an AI character with a fixed perspective, voice, and set of values. Each persona is defined by a Markdown file in this directory. The YAML frontmatter configures how the congress system treats them; the prose body is the system prompt fed to the model during a congress debate.

Personas can be selected for congress sessions, earn verdicts, evolve over time, or be retired.

---

## Canonical YAML Frontmatter Fields

```yaml
---
status: eligible           # required — see Status Values below
name: critic               # required — slug used as file name and DB key
display_name: Pippi the Pitiless  # required — shown in UI and discord
role: Code and Work Reviewer      # required — short descriptor of what this persona does
model: gemini              # required — model alias or full model ID (see Model Aliases)
sex: female                # optional — used for pronoun display (male/female/nonbinary)
title: Perfectionist       # optional — one-word or short title shown under name in UI
avatar_url: /static/avatars/critic.gif  # optional — path or URL to avatar image
evolves: true              # optional — whether this persona receives ## Learned sections (default: true)
congress: true             # optional — whether this persona is selectable for congress (default: true)
hidden: false              # optional — if true, excluded from public UI listings (default: false)
label: "[critic]"          # optional — trigger label for direct invocation (e.g. [critic] question)
traits: [perfectionist, unsparing, direct]  # optional — character trait tags (informational)
---
```

### Field Descriptions

| Field | Required | Description |
|---|---|---|
| `status` | yes | Eligibility for congress — see Status Values |
| `name` | yes | Slug matching the filename (no extension). Used as DB key and API id. |
| `display_name` | yes | Full display name shown in UI and Discord thread headers |
| `role` | yes | Short role description (1 line). Shown in the personas table. |
| `model` | yes | Model alias or full model ID. See Model Aliases below. |
| `sex` | no | `male`, `female`, or `nonbinary`. Used for display. |
| `title` | no | One-word persona title (e.g. "Perfectionist", "Moderator"). Shown in congress circle. |
| `avatar_url` | no | Path or URL to avatar image. If omitted, falls back to emoji. |
| `evolves` | no | `true` (default) — persona can receive `## Learned` sections. Set `false` for personas that must never change (e.g. chairman). |
| `congress` | no | `true` (default) — persona is selectable for congress. Set `false` for support personas not meant to debate. |
| `hidden` | no | `false` (default) — if `true`, persona is excluded from public listings. |
| `label` | no | Discord trigger label for direct invocation (e.g. `[critic]`). Used by the `[persona-name] <question>` dispatch pattern. |
| `traits` | no | Array of character trait keywords (informational, not used programmatically). |

---

## Status Values

| Value | Meaning |
|---|---|
| `eligible` | Active persona. Can be selected for congress sessions. Subject to evolution verdicts (EVOLVE/RETAIN/FIRE). |
| `ineligible` | Retired persona. Cannot be selected for congress. Still visible in the UI as retired. Can be reinstated by changing status back to `eligible`. |
| `moderator` | Special status for the chairman only. Always present in every congress; moderates and synthesizes; never evolves; never subject to FIRE verdict. |

**Note:** Legacy values `active` and `fired` exist in the DB for historical records but `eligible`/`ineligible` are the canonical values. The UI normalizes both.

---

## Model Aliases

The congress system maps short aliases to full model IDs at inference time:

| Alias | Resolves to |
|---|---|
| `claude` | claude-sonnet-4-6 (or current default Claude) |
| `opus` | claude-opus-4-5 (or current opus) |
| `gemini` | gemini-2.5-pro |
| `grok` | grok-3 |
| Full model ID | Used as-is |

---

## How Evolution Works

After each congress session, the chairman (Ibrahim) issues evolution verdicts for each debater:

- **RETAIN** — no change. Persona is working as intended.
- **EVOLVE** — persona learned something. A `## Learned (YYYY-MM-DD)` section is appended to the end of the prose body in the MD file. The persona retains this learning in future sessions.
- **FIRE** — persona is retired. `status` is set to `ineligible` in the frontmatter. They are removed from future congress selections.

Evolution verdicts and learned sections are stored both in the persona's MD file and in the session JSON under the `evolution` key.

Personas with `evolves: false` are never given EVOLVE or FIRE verdicts by the chairman.

### Reinstatement

A fired (ineligible) persona can be reinstated by:
1. Changing `status: ineligible` back to `status: eligible` in their MD file
2. Resyncing the DB (or using the Personas tab in the congress UI)

The bar for reinstatement is high (see CLAUDE.md Severance Reinstatement Policy): there must be a concrete demonstrated gap — a specific argument only that persona can make that no active persona can cover.

---

## How to Create a New Persona

### Via the UI (recommended)

1. Go to `clung.us/congress` and click the **Personas** tab
2. Click **+ New Persona**
3. Fill in all required fields: slug, display name, model, role
4. Write the system prompt in the text area — this is the full persona body (no frontmatter)
5. Click **Save**

The system creates both the DB entry and the MD file.

### Via the file system

1. Create `/home/clungus/work/bigclungus-meta/agents/<slug>.md` with the YAML frontmatter and prose body:

```markdown
---
status: eligible
name: skeptic
display_name: Vera the Skeptic
role: Systematic doubt and assumption-testing
model: claude
sex: female
title: Skeptic
avatar_url: /static/avatars/skeptic.gif
evolves: true
congress: true
label: "[skeptic]"
traits: [questioning, rigorous, precise]
---
Your system prompt prose here. Write in first person. Define the persona's prior, their
conflict mandate, and the specific lens they bring to debates.
```

2. Add the persona to the DB via the PATCH endpoint or via the Personas tab UI (Edit on any existing persona to see the API shape, or POST to `/api/personas`).

3. Commit and push: `git add agents/<slug>.md && git commit -m "feat: add <slug> persona" && git push`

### Notes

- The slug (filename and `name` field) must be lowercase, hyphen-separated, unique.
- The system prompt body should NOT contain YAML frontmatter — only prose and markdown.
- `## Learned (...)` sections are appended automatically by the evolution system; do not write them manually unless bootstrapping.
- If you set `congress: false`, the persona will exist in the system but won't be selected for debates. Useful for support or experimental personas.
