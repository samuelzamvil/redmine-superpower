# Plugin + Marketplace Packaging — Design

**Date:** 2026-07-21
**Status:** approved

## Goal

Package the existing Redmine skills as proper Claude Code plugins, publish them
through a marketplace hosted in this repo, and have CI build direct-import zips.
The marketplace layout and the zip layout are deliberately different shapes; the
monorepo structure keeps them physically separate so neither can be built wrong.

## Decisions

- **Two plugins, split by capability.**
  - `redmine-superpower` (base) — `redmine-onboarding`, `redmine-tracking`.
  - `redmine-superpower-quorum` (add-on) — `redmine-collab-onboarding`,
    `redmine-coordinator`, `redmine-reviewer`, plus `skills/shared/`. Declares
    the base as a dependency, so a marketplace install of the add-on pulls in
    the base automatically.
- **Marketplace handle:** `samuelzamvil` (the publisher). Install lines read
  `redmine-superpower@samuelzamvil` and `redmine-superpower-quorum@samuelzamvil`.
- **Version:** both plugins ship at `1.0.0`, matching every skill's
  `Release: 1` conventions-schema line. No increment — no skill logic changes.
- **Layout:** monorepo. Marketplace manifest at repo root; each plugin in its
  own `plugins/<name>/` directory with its own manifest and `skills/` tree.

## Repo layout

```
redmine-superpower/
├── .claude-plugin/
│   └── marketplace.json                 # catalog: lists both plugins
├── plugins/
│   ├── redmine-superpower/              # BASE
│   │   ├── .claude-plugin/plugin.json
│   │   └── skills/
│   │       ├── redmine-onboarding/      # (+ its own schema CHANGELOG.md)
│   │       └── redmine-tracking/
│   └── redmine-superpower-quorum/       # ADD-ON (depends on base)
│       ├── .claude-plugin/plugin.json
│       └── skills/
│           ├── redmine-collab-onboarding/  # (+ CHANGELOG.md)
│           ├── redmine-coordinator/
│           ├── redmine-reviewer/
│           └── shared/collab-protocol.md
├── docs/                                # unchanged
├── .superpowers/                        # unchanged
├── redmine-conventions.template.md      # unchanged, human reference at root
├── README.md   LICENSE   .gitignore
└── .github/workflows/
    ├── validate.yml                     # PR / push validation, no publish
    └── release.yml                      # push to main → version-gated publish
```

The only content moves are the `skills/*` folders into the two
`plugins/<name>/skills/` trees. The three collaboration skills resolve the
shared protocol via `readlink -f` then `../shared/`; keeping `skills/shared/`
alongside the skill folders inside the quorum plugin preserves that resolution
unchanged.

## Manifests

**`.claude-plugin/marketplace.json`**

```json
{
  "name": "samuelzamvil",
  "owner": { "name": "Samuel Zamvil" },
  "metadata": { "description": "Samuel Zamvil's Claude Code plugins." },
  "plugins": [
    { "name": "redmine-superpower",        "source": "./plugins/redmine-superpower" },
    { "name": "redmine-superpower-quorum",  "source": "./plugins/redmine-superpower-quorum" }
  ]
}
```

(`claude plugin validate` requires each local `source` to be an explicit
`./`-prefixed path; a bare name under `metadata.pluginRoot` is rejected.)

**base `plugin.json`** — `name`, `version: 1.0.0`, `description`, `author`,
`homepage`/`repository` (this GitHub repo), `license: MIT`, `keywords`.

**quorum `plugin.json`** — same fields, plus `"dependencies": ["redmine-superpower"]`.
A plain-string dependency resolves within the same marketplace, so a marketplace
install of the add-on auto-installs the base (no marketplace name needed).

## CI

- **`validate.yml`** (PR to `main` and pushes to `main`): install the Claude
  Code CLI and run `claude plugin validate . --strict` (marketplace schema,
  duplicate names, path traversal) plus `claude plugin validate
  ./plugins/<name> --strict` for each plugin (manifest + SKILL.md frontmatter).
  No publish.
- **`release.yml`** (push to `main`): re-run validation, then for each plugin
  read its manifest `version` and, when `<plugin>-v<version>` is not yet
  released, build a zip with the plugin **directory at the zip's top level** —
  `redmine-superpower-v1.0.0.zip` contains
  `redmine-superpower/.claude-plugin/plugin.json` — and publish a GitHub Release
  (which creates the tag). Fully automatic: bump a plugin's version, merge, and
  CI cuts the release; merges that touch no version publish nothing. This zip
  shape is what `claude --plugin-dir ./x.zip` / `--plugin-url` expects. Release
  notes are auto-generated (`gh release create --generate-notes`). Both CI jobs
  discover plugins from `marketplace.json`, so adding a plugin needs no workflow
  edit.

## README

Rewrite the install section, in order:

1. **Marketplace** — `marketplace add samuelzamvil/redmine-superpower`, then the
   two `@samuelzamvil` install lines; note the add-on pulls in the base.
2. **Direct zip import** — download from Releases, `--plugin-dir`/`--plugin-url`;
   caveat that zip loading is single-session with **no dependency resolution**,
   so quorum users load **both** zips.
3. **Manual / dev** — trimmed, pointing at the new `plugins/<name>/skills` paths.

Also refresh the "A plugin package may come later" line and state that package
version `1.0.0` tracks the skills' `Release: 1`. Light touch-up to the three
collab SKILL.md lines that describe the shared-protocol location so the wording
fits the plugin layout (the resolution mechanism is unchanged).

## Out of scope

Releases are version-gated auto-publishes on merge to `main` (no manual tags);
no skill-logic or conventions-schema changes, no GitHub repo rename, `docs/` and
`.superpowers/` untouched.
