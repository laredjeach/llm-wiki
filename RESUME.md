# 🔧 RESUME HERE — WIP Handoff (wiki `stacks` feature)

> **Temporary working note.** Delete this file once the `stacks` work is committed,
> reviewed, and merged. It captures session context that would otherwise be lost on
> restart. Last updated: 2026-06-20.

---

## TL;DR — where we left off

We added a new first-class compiled wiki category, **`stacks`** (`category: stack` →
`wiki/stacks/`), sibling to `concepts/topics/references/theses`, plus a new
`/wiki:ingest-stack` command. **All functional tests pass.** Nothing is committed yet —
everything below is in the working tree on branch `master`.

**The single most important next step:** review/refine `claude-plugin/commands/ingest-stack.md`
(never shown to the user yet), then commit on a feature branch, then push.

---

## Git state at handoff

- Branch: `master` (clean base commit `75623c6 Format v0.12 changelog bullets`)
- **Nothing staged or committed this session.** ~52 tracked files modified, 5 new untracked (ours).
- Plugin version still `0.12.0` — **not bumped** (see Open Items).

### New (untracked) files — ours
```
claude-plugin/commands/ingest-stack.md                     # /wiki:ingest-stack (NEEDS REVIEW)
tests/fixtures/golden-wiki/wiki/stacks/_index.md           # stacks index
tests/fixtures/golden-wiki/wiki/stacks/template-stack.md   # superset template (lint-clean)
tests/fixtures/golden-wiki/inbox/.gitkeep                  # fixes pre-existing fixture bug
tests/fixtures/golden-wiki/inbox/.processed/.gitkeep       # fixes pre-existing fixture bug
```

### Modified (by area)
- **Spec:** `AGENTS.md`, `claude-plugin/skills/wiki-manager/references/wiki-structure.md`,
  `claude-plugin/skills/wiki-manager/references/linting.md`,
  `claude-plugin/skills/wiki-manager/SKILL.md`, `claude-plugin/commands/wiki.md`
- **Local linter:** `scripts/llm-wiki` (4 stack edits), `scripts/sync-codex-plugin.sh` (router line)
- **Tests:** `tests/test-structure.sh`, golden `_index.md` files, **36 regenerated defect fixtures**
- **Generated mirrors (do NOT hand-edit):** `plugins/llm-wiki/**`, `plugins/llm-wiki-opencode/**`
- `README.md` (modified — verify whether intentional before committing)

---

## What changed and why

1. **`stacks` is a superset schema, not a new file kind.** A stack page is a normal wiki
   article with `category: stack` PLUS extra fields. We deliberately reused the existing
   placement/lint machinery instead of a `type:`-keyed kind (like theses).
   - Required (standard article) fields: `title, category: stack, sources, created, updated, tags, confidence, summary`
   - Extra stack fields: `topic, date, scenario, llm_usage, stack_type, related_topics`
   - Body: `## Problem Context` · `## Problem-Oriented Notes` (`### Key Decisions`) ·
     `## Normalized Stack Table` · `## Comparison & Ratings Table` · `## Stack Overview` · `## LLM Hints`

2. **Two linters both had to learn `stack`:**
   - Spec linter (`references/linting.md`): C11 placement map (`stack → wiki/stacks/`),
     C12 allowlist (`stacks/`), C13 category-inference line.
   - Deterministic CLI (`scripts/llm-wiki`): `ARTICLE_CATEGORIES`, `ARTICLE_DIRS`,
     `WIKI_ALLOWED`, and the `ensure_dir_index` targets list — 4 minimal edits, no exemptions
     (user explicitly did not want template-exemption logic).

3. **Template is fully lint-clean** by mirroring `sample-concept.md`: non-empty `summary`/`tags`,
   `sources: [raw/articles/2026-01-01-sample-article.md]` (a real fixture file), `volatility: warm`.

4. **`inbox/.gitkeep` placeholders fix a PRE-EXISTING bug, unrelated to stacks:** the golden
   fixture's `inbox/` is an empty dir git can't track, so it vanished on checkout and made
   `test-local-cli-lint.sh` fail on any fresh clone (unconditional inbox check at
   `scripts/llm-wiki:545-552`). Proven against pristine `git archive HEAD`.

5. **Router:** `ingest-stack` routing hint was added to the source `SKILL.md` and the
   `sync-codex-plugin.sh` replacement list (aligned with our work; keep it).

---

## Test status

| Test | Result |
|------|--------|
| `test-plugin-validate.sh` | ✅ 96/96 |
| `test-structure.sh` | ✅ 183/183 |
| `test-local-cli-lint.sh` | ✅ 22/22 |
| `test-session-capture.sh` | ✅ 17/17 |
| `test-codex-sync.sh` / `test-opencode-sync.sh` | ⏳ RED **only as commit-guards** (`git diff --quiet HEAD -- plugins/`). Mirrors are correct; they go **green once `plugins/` is committed.** |
| promptfoo router eval | ⚠️ NOT run — recommended for the new `ingest-stack` routing (costs ~$2-5, needs `ANTHROPIC_API_KEY`) |
| `test-codex-runtime.sh` | ⚠️ NOT run — optional, only when touching Codex packaging |

To re-verify after restart:
```bash
./tests/test-plugin-validate.sh && ./tests/test-structure.sh && ./tests/test-local-cli-lint.sh
# sync tests only pass once committed:
./scripts/sync-codex-plugin.sh && ./scripts/sync-opencode-plugin.sh
./tests/test-codex-sync.sh && ./tests/test-opencode-sync.sh
```

---

## Open items / next steps (in order)

1. **Review & refine `claude-plugin/commands/ingest-stack.md`** — the LLM instructions were
   scaffolded but never shown/refined. This is the main unfinished design work.
2. **Bump version + changelog?** Adding a command + category likely warrants a version bump
   and README/changelog entry per `.claude/release-checklist.md`. Not done.
3. **Commit** on a feature branch (we're on `master`, the default — branch first):
   `git switch -c feat/wiki-stacks`, stage stack changes + regenerated `plugins/`, commit.
   This greens the two sync tests.
4. **Push (GitHub).** Per `CLAUDE.md`: use `gh` web login + HTTPS, not SSH:
   ```bash
   gh auth login --web --git-protocol https   # if needed
   git -c credential.helper='!gh auth git-credential' \
     push https://github.com/nvk/llm-wiki.git feat/wiki-stacks
   ```
   (Then open a PR — do NOT push straight to `master` unless that's the intent.)
5. **Planned repo move (NOT done):**
   `/Users/jaredleach/Project library/Tools/llm-wiki` → `/Users/jaredleach/Documents/Projects/llm-wiki`
   via a single `mv`. Safe (carries `.git` + untracked). Will invalidate the session CWD —
   restart Claude rooted at the new path afterward.

---

## Gotchas to remember

- **Never hand-edit** `plugins/llm-wiki/**` or `plugins/llm-wiki-opencode/**` — generated.
  Edit `claude-plugin/skills/...`, then re-run both sync scripts.
- `AGENTS.md` was also modified by the user/linter (the `stacks` tree line) — intentional, don't revert.
- Related personal notes were moved OUT of this repo to
  `/Users/jaredleach/Desktop/Alias-TO-DOs/search-tooling-session-notes.md` (has a back-pointer).
- Repo auth: GitHub CLI web login + HTTPS transport, never SSH (see `CLAUDE.md`).
