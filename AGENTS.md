# Repository Guidelines

## What this repo is

A fork of [mihomo-party-org/override-hub](https://github.com/mihomo-party-org/override-hub). Contains Mihomo Party override scripts/configs consumed directly via raw GitHub URLs. No build step, no runtime, no package manager.

- `javascript/`: JS overrides. Each file must export a `function main(config)` that mutates and returns the config object.
- `yaml/`: YAML overrides and rule sets.
- `README.md`: lists raw URLs for each file. **Update when adding or renaming files** — users paste these links directly into Mihomo Party.

## Validation (no automated tests)

```sh
git diff -- <path>                       # review patch before commit
node --check javascript/<file>.js        # syntax check only; no test runner exists
```

YAML: visually verify 2-space indentation and that `path`/`url` values in rule-providers still resolve.

## Coding style

- **JS**: 2-space indent, trailing commas where already present, object-heavy style. Do not reformat existing files.
- **YAML**: 2-space indent, stable key ordering. Avoid unnecessary quote changes.
- **Filenames**: Chinese names are intentional (e.g. `布丁狗的订阅转换.js`). Do not rename them — raw URLs in README embed percent-encoded versions of these names.

## Fork vs upstream

This fork has two permanent branches:
- `main` — fork's own branch; fork-specific files live here (e.g. `正则匹配设置代理组图标.js` is not in upstream README).
- `upstream-main` — tracks `upstream/main` locally.

Upstream sync is **automated**: `.github/workflows/sync-upstream.yml` runs daily at 09:17 CST, merges `upstream/main` into `main`, and opens a PR on `automation/sync-upstream-main`. Manual upstream sync only needed if you can't wait for the schedule:

```sh
git fetch upstream
git switch main
git merge --no-ff upstream/main
```

## Commit messages

Prefer Conventional Commits: `fix: ...`, `feat: ...`, `chore: ...`. Plain imperative subjects are also acceptable. Match the existing log style.

## PR conventions

- Short summary of behavior change + affected files.
- Note any manual validation done (e.g. "syntax checked with `node --check`").
- Raw URL links in README must use the **upstream org path** (`mihomo-party-org/override-hub`) for files that exist upstream; fork-only files use the fork's raw URL (`r404r/override-hub`).
