# 🔧 RESUME HERE — Handoff (wiki `stacks` feature)

> **Working handoff note** for resuming in Claude Code, Cursor, or any agent. Captures
> session context that would otherwise be lost on restart. Delete once the `stacks` work
> is reviewed and merged upstream. **Last updated: 2026-06-20** (post-commit reconcile).

---

## TL;DR — current state

We added a new first-class compiled wiki category, **`stacks`** (`category: stack` →
`wiki/stacks/`), sibling to `concepts/topics/references/theses`, plus a new
`/wiki:ingest-stack` command, and fully wired it into routing, the skill, docs, and
mirrors. **All structural tests pass. Everything is committed and pushed.**

- **Branch:** `feat/wiki-stacks` — **working tree CLEAN**, nothing uncommitted.
- **HEAD:** `774f44c Note fork provenance and push status in RESUME`
  on top of `2f54461 Add wiki stacks category and /wiki:ingest-stack command`
  (base `75623c6 Format v0.12 changelog bullets`). 2 commits ahead of `master`.
- **Pushed:** `origin/feat/wiki-stacks` is in sync with local HEAD (fork `laredjeach/llm-wiki`).
- **Plugin version:** still `0.12.0` — **not bumped** (open decision, see below).

**Single most important next step:** review/refine the LLM instructions in
`claude-plugin/commands/ingest-stack.md` — it was scaffolded and wired, but the body
prose has not had a deliberate design pass. Then decide on version bump + upstream PR.

---

## Provenance / remotes

- This working copy is a **fork**: `origin` → `https://github.com/laredjeach/llm-wiki.git`
  (where we push), `upstream` → `https://github.com/nvk/llm-wiki.git` (parent).
- Open a PR upstream when ready:
  `gh pr create --repo nvk/llm-wiki --head laredjeach:feat/wiki-stacks`
- Auth: GitHub CLI web login + **HTTPS** transport, never SSH (see `CLAUDE.md`).

---

## What's in the feature (committed)

### The `stacks` category — superset schema, not a new file kind
A stack page is a normal wiki article with `category: stack` PLUS extra fields, so it
reuses the existing placement/lint machinery (no `type:`-keyed kind like theses).
- Standard article fields: `title, category: stack, sources, created, updated, tags, confidence, summary`
- Extra stack fields: `topic, date, scenario, llm_usage, stack_type, related_topics`
- Body: `## Problem Context` · `## Problem-Oriented Notes` (`### Key Decisions`) ·
  `## Normalized Stack Table` · `## Comparison & Ratings Table` · `## Stack Overview` · `## LLM Hints`

### Both linters learned `stack`
- Spec linter (`references/linting.md`): C11 placement map, C12 allowlist, C13 inference.
- Deterministic CLI (`scripts/llm-wiki`): `ARTICLE_CATEGORIES`, `ARTICLE_DIRS`,
  `WIKI_ALLOWED`, `ensure_dir_index` targets — 4 minimal edits, no template-exemption logic.

### `/wiki:ingest-stack` command + full wiring
- `claude-plugin/commands/ingest-stack.md` — the command (⚠️ body prose still needs a review pass).
- **Router** (`claude-plugin/commands/wiki.md`): added priority **`0c | Stack`** NL-routing row
  (mirrors how `0 | Collection Ingest` exposes `/wiki:ingest-collection`) + a post-init suggestion.
- **Skill** (`SKILL.md`): added a first-class **`### Stack Pages`** workflow entry.
- **Codex sync** (`scripts/sync-codex-plugin.sh`): added an `ingest-stack` line to the
  Codex-specific Workflows replacement block (Codex rewrites that section, so the source
  SKILL edit alone doesn't reach it).
- **Docs:** `AGENTS.md` (operations list + full `### Ingest Stack` section), `README.md`
  (structure tree `stacks/` line, command-table rows, usage examples).
- **Eval:** `tests/promptfooconfig.yaml` — added a stack-routing test case.
- **Generated mirrors** regenerated + committed: `plugins/llm-wiki/**`, `plugins/llm-wiki-opencode/**`.

### Fixtures
- `tests/fixtures/golden-wiki/wiki/stacks/_index.md`, `template-stack.md` (lint-clean superset template).
- `inbox/.gitkeep` + `inbox/.processed/.gitkeep` — fix a PRE-EXISTING fixture bug (empty `inbox/`
  vanished on fresh checkout, breaking `test-local-cli-lint.sh`). Unrelated to stacks.
- 36 regenerated defect fixtures.

---

## Test status (last run this session)

| Test | Result |
|------|--------|
| `test-plugin-validate.sh` | ✅ 96/96 |
| `test-structure.sh` | ✅ 183/183 |
| `test-local-cli-lint.sh` | ✅ 22/22 |
| `test-session-capture.sh` | ✅ 17/17 (per prior run) |
| `test-codex-sync.sh` / `test-opencode-sync.sh` | ✅ green now that `plugins/` is committed (they only RED as commit-guards on uncommitted mirror drift) |
| promptfoo router eval (stack case) | ⚠️ **Could not validate locally** — the eval workspace has no resolvable wiki hub, so the `/wiki` router replies "I couldn't find your wiki" and bails before dispatching to ANY skill. Confirmed by running the existing `ingest-collection` routing case as a baseline — it fails identically. Pre-existing harness limitation, not a defect in our routing. CI provides a resolvable hub. (Also note: running it locally needs `@anthropic-ai/claude-agent-sdk`, which is not a saved dependency.) |
| `test-codex-runtime.sh` | ⚠️ NOT run — optional, only when touching Codex packaging |

Re-verify after restart:
```bash
./tests/test-plugin-validate.sh && ./tests/test-structure.sh && ./tests/test-local-cli-lint.sh
```

---

## Open items / next steps (in order)

1. **Review & refine `claude-plugin/commands/ingest-stack.md` body prose** — the main
   unfinished design work. Everything around it (routing, schema, lint, mirrors) is done.
2. **Decide on version bump + changelog** — adding a command + category likely warrants
   a bump and a README/changelog entry per `.claude/release-checklist.md`. Not done.
3. **Open upstream PR** to `nvk/llm-wiki` when the command body is reviewed (command above).
4. **Repo move — ✅ DONE (2026-06-20).** Repo now lives at
   `/Users/jaredleach/Documents/Projects/llm-wiki` (moved from `…/Project library/Tools/llm-wiki`).
   Git, remotes, and untracked files all carried over cleanly. The Claude project state
   dir (memory + transcripts) was migrated alongside to the matching new slug
   `~/.claude/projects/-Users-jaredleach-Documents-Projects-llm-wiki/`.
5. **Brain-Master integration (DEFERRED — do not start unprompted):** the user intends to
   eventually wire llm-wiki (`/wiki:ingest` / `/wiki:ingest-stack`) as an easy ingestion
   trigger for their **Brain-Master vault** at `/Users/jaredleach/Documents/Brain-Master`.
   Decisions are on hold until the user finishes
   their other systems; they will later use Claude Code + Cursor to cross-reference, detect
   conflicts, and design how to merge / connect / guard the systems. **Hold for explicit go-ahead.**
6. **HUB not yet created** — no `~/.config/llm-wiki/config.json`, no `~/wiki/`. A "research"
   topic-wiki init was requested but PAUSED pending the user's choice of HUB location.

---

## Where context lives (for handoff to Cursor / another agent)

- **This file (`RESUME.md`)** — the human-readable handoff. Cursor can read it directly.
- **In-repo project docs:** `README.md` (public story + changelog), `AGENTS.md` (deepest spec),
  `CLAUDE.md` (dev/test/sync rules for this repo), `.claude/release-checklist.md`.
- **Plugin source of truth:** `claude-plugin/commands/*.md`, `claude-plugin/skills/wiki-manager/`
  (`SKILL.md` + `references/*.md`). Generated mirrors under `plugins/` — never hand-edit.
- **Out-of-repo Claude memory (NOT visible to Cursor):**
  `/Users/jaredleach/.claude/projects/-Users-jaredleach-Documents-Projects-llm-wiki/memory/`
  → `brain-master-integration-intent.md` (the deferral above) + `MEMORY.md` index.
  Key facts are duplicated into this RESUME.md so Cursor isn't blind to them.
- **Parked personal side-thread (NOT part of llm-wiki):**
  `/Users/jaredleach/Desktop/Alias-TO-DOs/search-tooling-session-notes.md` — `rg`/`rga` vs
  semantic search teaching notes + a paused "find MYST brand/Shopify files" task. `rga`
  (ripgrep-all) was installed via Homebrew during that thread.

---

## Gotchas to remember

- **Never hand-edit** `plugins/llm-wiki/**` or `plugins/llm-wiki-opencode/**` — generated.
  Edit `claude-plugin/skills/...`, then re-run BOTH sync scripts and commit `plugins/`.
- The Codex SKILL.md `## Workflows` section is **rewritten** by `sync-codex-plugin.sh`, not
  copied — new workflow entries must be added to that script's replacement block too.
- Sync tests (`test-codex-sync.sh` / `test-opencode-sync.sh`) compare `plugins/` against
  committed `HEAD`; they fail on any uncommitted mirror drift and pass once committed.
- Local-only env: `@anthropic-ai/claude-agent-sdk` was installed with `--no-save` to run the
  promptfoo eval (lives only in gitignored `node_modules`; `package.json` untouched).
