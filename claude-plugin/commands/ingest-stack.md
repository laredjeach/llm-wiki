---
description: "Build a stack page: research or assemble the layered tools/frameworks for a scenario and compile a structured stack article into wiki/stacks/."
argument-hint: "<scenario|topic|url|filepath> [--title \"Title\"] [--stack-type <type>] [--wiki <name>] [--local] [--new-topic <name>] [--from-sources] [--include-archived]"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash(ls:*), Bash(wc:*), Bash(date:*), Bash(mkdir:*), Bash(basename:*), WebFetch, WebSearch
---

## Your task

Compile a **stack page** — a structured wiki article in `wiki/stacks/` that
documents how a coherent set of layered or interlocking components (tools,
frameworks, services) fits together to address a scenario. A stack page is a
compiled article with `category: stack`; it is synthesis, not a raw dump.

**Resolve the wiki.** Do NOT search the filesystem or read reference files first —
follow these steps:
1. Read `$HOME/.config/llm-wiki/config.json`. If it has `hub_path`, expand the
   leading `~` only (not tildes in `com~apple~CloudDocs`) and prefer that path;
   use `resolved_path` only as a fallback cache when the expanded `hub_path` is
   unavailable and `resolved_path` is initialized. If config has only
   `resolved_path`, use it. If the configured path can be statted but reading
   `wikis.json` or listing `topics/` fails with `Operation not permitted`, stop
   and ask the user to grant Full Disk Access/iCloud Drive access; do not fall
   back to `~/wiki` or `resolved_path`. Do not write machine-specific
   `resolved_path` into shared configs.
2. If no config → read `$HOME/wiki/_index.md`. If it exists → HUB = `$HOME/wiki`.
   If nothing found, ask the user where to create the wiki.
3. **Wiki location** (first match): `--local` → `.wiki/` in CWD; `--wiki <name>` →
   `HUB/wikis.json` lookup with portable path resolution (`<HUB>`, `~`, absolute,
   or HUB-relative); if the registry path is stale, fall back to
   `HUB/topics/<name>`; CWD has `.wiki/` → use it; else → HUB.
4. Read `<wiki>/_index.md` to verify. If missing and `--new-topic <name>` is set →
   create the topic wiki first (full init protocol). If missing and no
   `--new-topic` → stop with "No wiki found. Use `--new-topic <name>` to create
   one, or run `/wiki init` first."

Read the structure spec at `skills/wiki-manager/references/wiki-structure.md`
(§ "Stack Pages") and the compilation protocol at
`skills/wiki-manager/references/compilation.md` for conventions. Then build the
stack page.

### Archive awareness

Do not write into archived topic wikis by default. If `--wiki <name>` resolves to
`status: archived` or a path under `topics/.archive/`, stop and ask the user to
restore it with `/wiki:archive restore <name>` or rerun with
`--include-archived`. When explicitly included, write only inside that archived
topic path and keep it archived.

### Parse $ARGUMENTS

- **Source/scenario**: the first non-flag argument — a scenario description
  ("realtime analytics dashboard for IoT data"), a topic, a URL, or a file path.
- **--title "Title"**: override the derived stack title.
- **--stack-type <type>**: set the `stack_type` field (e.g. `web-app`,
  `data-pipeline`, `agent`, `research`). Inferred from the scenario if omitted.
- **--from-sources**: assemble the stack only from sources already in the target
  wiki's `raw/` (no web research). Default is to research gaps when the wiki is
  thin on the scenario's components.
- **--new-topic <name>**, **--wiki <name>**, **--local**, **--include-archived**:
  as in `/wiki:ingest`.

### Build flow

1. **Frame the scenario.** Restate the problem the stack solves in one or two
   sentences. Derive `topic`, `scenario`, and `stack_type`. Identify the role
   groups the stack needs (e.g. ingestion, storage, processing, serving, UI,
   orchestration, observability) — role groups are the rows of the stack tables.
2. **Gather components.** Survey existing `raw/` sources first (read indexes,
   don't scan blindly). With `--from-sources`, stop here and synthesize only from
   what exists. Otherwise run targeted `WebSearch`/`WebFetch` to fill gaps: for
   each role group, identify the leading tool/framework options, their key
   features, and canonical site/docs/GitHub links. Prefer primary sources.
3. **Normalize.** Collapse aliases and reposts; pick one canonical entry per
   (role group, tool). Note source type (docs, repo, vendor page, paper, etc.).
4. **Compare & rate.** For each candidate, give a comparison rating and a short
   justification grounded in the gathered evidence — not vibes. Be explicit about
   tradeoffs and when to pick each option.
5. **Write the stack page** to `wiki/stacks/<slug>.md` using the schema and body
   from `wiki-structure.md` § "Stack Pages":
   - Frontmatter superset: `title`, `category: stack`, `sources` (raw/ paths used),
     `created`, `updated`, `confidence`, `summary`, plus `topic`, `date`,
     `scenario`, `llm_usage`, `tags`, `stack_type`, `related_topics`.
   - Body: `## Problem Context`, `## Problem-Oriented Notes` (with
     `### Key Decisions`), `## Normalized Stack Table`, `## Comparison & Ratings
     Table`, `## Stack Overview`, `## LLM Hints`.
   - Fill `## LLM Hints` with guidance for an agent operating this stack (gotchas,
     defaults, what to reach for first). Set `llm_usage` to how an LLM fits in.
6. **Cross-link.** Add dual-links to related concepts/topics/stacks and populate
   `related_topics`. Where a component already has a concept article, link it.
7. **Provenance.** If the web research surfaced sources worth keeping, ingest the
   best ones into `raw/` (via the ingest protocol) and reference them in
   `sources:`. Do not cite uningested web pages as the article's evidence base.

### Slug & indexes

1. Slug: `descriptive-slug.md` (no date — stack pages are living documents).
2. Write to `wiki/stacks/<slug>.md`. Set `confidence` honestly (`medium` unless
   corroborated by strong primary sources).
3. Update `wiki/stacks/_index.md`, `wiki/_index.md`, and the master `_index.md`.
4. Append to `log.md`: `## [YYYY-MM-DD] compile | Stack: Title (wiki/stacks/slug.md)`.
5. Report: the stack title, role groups covered, top picks per role group,
   sources used/ingested, and any gaps to research next.
