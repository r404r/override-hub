# Repository Guidelines

## Project Structure & Module Organization
This repository is a small content repo for Mihomo Party override examples.

- `javascript/`: JavaScript override scripts. Each file should export logic through the existing `main(config)` pattern.
- `yaml/`: YAML override examples and rule sets.
- `.github/workflows/`: repository automation, including upstream sync.
- `README.md`: user-facing usage instructions and raw-link examples.

Keep related changes together. If you update a script or YAML example that changes behavior, also update `README.md` when the public usage surface changes.

## Build, Test, and Development Commands
There is no build system or automated test suite in this repo. Use lightweight checks before committing:

- `rtk git status --short --branch`: confirm the intended files changed.
- `rtk sed -n '1,160p' javascript/<file>.js`: review JS formatting and structure.
- `rtk sed -n '1,120p' yaml/<file>.yaml`: review YAML indentation and keys.
- `rtk git diff -- <path>`: verify the final patch before commit.

For upstream maintenance:

- `rtk git fetch upstream`
- `rtk git switch upstream-main`
- `rtk git merge --ff-only upstream/main`

## Coding Style & Naming Conventions
Match the existing file style instead of reformatting aggressively.

- JavaScript: 2-space indentation, trailing commas where already used, preserve the current object-heavy style.
- YAML: 2-space indentation, stable key ordering when practical, avoid unnecessary quoting changes.
- Filenames: keep descriptive names in Chinese when they match the current repository style, for example `javascript/布丁狗的订阅转换.js`.

## Testing Guidelines
Validation is manual. Contributors should:

- check JavaScript for syntax mistakes and accidental key renames,
- check YAML for indentation and rule-path correctness,
- confirm referenced remote URLs and local `path` values still make sense,
- include a short validation note in the PR.

## Commit & Pull Request Guidelines
Prefer concise, imperative commit messages. The history currently mixes plain subjects and Conventional Commit prefixes; prefer the clearer form, such as `fix: update proxy domain` or `chore: sync upstream/main`.

PRs should include:

- a short summary of behavior changes,
- affected files or examples,
- any manual validation performed,
- screenshots only if UI-facing documentation changes.

For upstream contributions, branch from `upstream-main`. For fork-only features, branch from `main`.
