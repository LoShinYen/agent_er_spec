# Agent ER spec (`agent_er_spec`)

**English** · [繁體中文](README.zh-TW.md)

![ci](https://github.com/LoShinYen/agent_er_spec/actions/workflows/ci.yml/badge.svg)

A **Claude Code Skill** plus the spec it implements: turn SQL DDL or design Markdown into a long-lived **ER design bundle** (`.erd.json`), rendered into a draggable, zoomable interactive page.

Different from Mermaid `erDiagram`: this spec preserves **color-coded layers**, **multi-view diagrams**, **data-flow narratives**, and **design-decision logs** — the design semantics teams need for system docs, that Mermaid can't express.

## Two ways to use it

### A. As a spec / example library (raw)

Read the root files directly and produce bundles by hand or via an agent:

| Path | Purpose |
|------|---------|
| [erd-bundle.schema.json](erd-bundle.schema.json) | **JSON Schema** — authoritative structure for a bundle |
| [CLAUDE.md](CLAUDE.md) | Auto-loaded by Claude Code in this repo: invariants + JSON→host mapping |
| [examples/minimal.erd.json](examples/minimal.erd.json) | Smallest bundle (two tables, one edge) |
| [examples/ecommerce.erd.json](examples/ecommerce.erd.json) | 7 tables, 2 views, `dataFlows` + `designDecisions` |
| [examples/team.erd.json](examples/team.erd.json) | `0:1` / `0:N` showcase for optional relationships |
| [examples/demo.html](examples/demo.html) | Self-contained interactive page — opens offline |
| [sources/ecommerce.sql](sources/ecommerce.sql) · [sources/team.sql](sources/team.sql) | DDL inputs that derive the bundles above |

### B. As a Claude Code Skill (recommended)

[`plugins/er-bundle/skills/er-bundle/`](plugins/er-bundle/skills/er-bundle/) is a self-contained Skill: trigger description, schema, examples, validate + render scripts. Claude Code fires it when a user wants to produce or update an `.erd.json`.

**By default the skill produces both the `.erd.json` and an interactive HTML preview** (opens automatically). Tell it "JSON only" if you want to skip the preview. The [Quick start](#quick-start) CLI commands below are only needed when working with bundles outside Claude Code, or re-rendering after a manual edit.

**Install (pick one):**

- **Plugin marketplace** — add `.claude-plugin/marketplace.json` to your Claude Code plugin marketplace.
- **User-wide** — copy `plugins/er-bundle/skills/er-bundle/` to `~/.claude/skills/`. Fires in any project.
- **Project-local** (best while hacking on the Skill itself):
  ```bash
  mkdir -p .claude/skills && ln -sfn ../../plugins/er-bundle/skills/er-bundle .claude/skills/er-bundle
  ```
  `.claude/` is gitignored, so the symlink stays local; edits in `plugins/er-bundle/skills/er-bundle/` reflect instantly.

**Skill layout:**

```
plugins/er-bundle/skills/er-bundle/
├── SKILL.md                  # trigger description + workflow + anti-patterns
├── references/
│   ├── schema.json           # copy of root erd-bundle.schema.json (CI diffs to catch drift)
│   └── layout-heuristics.md  # rules for picking table coordinates
├── examples/                 # minimal, ecommerce, team (JSON + SQL), demo.html
├── scripts/
│   ├── validate.py           # JSON Schema + cross-checks
│   └── render_html.py        # bundle → standalone interactive HTML
└── tests/                    # 32 unit tests covering all of the above
```

## Spec highlights

A bundle requires `meta` / `layers` / `tables` / `diagrams`. Full surface in the [schema](erd-bundle.schema.json); the commonly-missed optional fields:

- **Column-level**: `nullable`, `default`, `onDelete` / `onUpdate`, `enumValues`
- **Table-level**: `tableConstraints[]` — composite PK/UQ, INDEX, CHECK (use `cols[*].tag` for single-column constraints)
- **Connection**: `cardinality` (`1:1` / `1:N` / `N:N` / `0:1` / `0:N`), `dashed` (logical relation, non-physical FK). `0:*` corresponds to nullable FK.
- **Optional sections**: `dataFlows` (write paths), `designDecisions` (old vs new)

## Quick start

```bash
# Validate (schema + cross-checks: FK endpoints, layer refs, positions coverage)
python3 plugins/er-bundle/skills/er-bundle/scripts/validate.py examples/ecommerce.erd.json

# Render to an interactive HTML page
python3 plugins/er-bundle/skills/er-bundle/scripts/render_html.py examples/ecommerce.erd.json -o /tmp/out.html
open /tmp/out.html
```

Both scripts need `jsonschema`: `pip install --user jsonschema`.

## Tests

```bash
python3 -m unittest discover -s plugins/er-bundle/skills/er-bundle/tests
```

CI runs on every push / PR (Python 3.11 + 3.12) — see [.github/workflows/ci.yml](.github/workflows/ci.yml).

## Relationship to a "full product page"

This repo only defines the **data shape** and a **minimal interactive UI**. A real product can layer additional pages on top (architecture notes, diff views) in the host project; `dataFlows` and `designDecisions` are reserved in the schema for that extension.

## License

[MIT](LICENSE)
