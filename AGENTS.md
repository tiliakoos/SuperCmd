# AGENTS.md — Nick's SuperCmd fork

Applies to any agent working in `/Users/NickT/Developer/personal/SuperCmd`.

This is a **fork**, not a repo we own. Upstream is `SuperCmdLabs/SuperCmd`; `origin` is `tiliakoos/SuperCmd`. `CLAUDE.md` in this repo is upstream's architecture doc and is the reference for how SuperCmd works. This file covers how *this fork* is worked in.

## Project memory lives in the Obsidian vault

**Read before starting work, and update when you finish:**

`/Users/NickT/Documents/Obsidian_Vault/Projects/04_supercmd/`

| Note | What it holds |
|---|---|
| `README.md` | Index, plus a 30-second orientation. |
| `fork-workflow.md` | **Read this first.** Part 1 is the mental model (three places, three branches, why). Part 2 is the procedure for any new fix/change/addition. Parts 3-5 are build, the four traps, and the PR checklist. |
| `status.md` | Current state and a reverse-chronological log of what changed. |
| `to-dos.md` | Open work: Fixes / Features / Maintenance / Improvements. |
| `decision-log.md` | Durable decisions and the reasoning behind them. |
| `product-brief.md` | What SuperCmd is and what Nick uses it for. |
| `technical/architecture.md` | Repo shape, command identity, test harness constraints. |
| `technical/raycast-import.md` | Raycast backup formats and the import path. |
| `future-enhancements.md` | Ideas parked on purpose. |

**When to write back:**

- Shipped something, or changed the installed build → add a dated entry at the top of `status.md`.
- Made a choice a future reader would otherwise re-litigate → `decision-log.md`, with the why and the cost accepted.
- Found or finished work → `to-dos.md`.
- Learned something durable about a format, an API, or a subsystem → a note under `technical/`.

Vault notes use YAML frontmatter (`created`, `updated`, `project`, `type`) and Obsidian wikilinks (`[[note-name]]`, no path). Match the existing style. The vault is project memory only; it never holds code.

## Branches

| Branch | Role |
|---|---|
| `main` | Pristine mirror of `upstream/main`. Never commit here. |
| `features/tiliakoos` | Daily driver. The installed app is built from this. Local-only changes live here. |
| `fix/...`, `feat/...` | One concern each, cut from `upstream/main`, pushed to origin, opened as PRs. |

## The rule for any new change

Ask: could this plausibly help everyone, or is it only for Nick?

- **Could help everyone** (a bug, a crash, a missing API): branch from `upstream/main`, commit, push to `origin`, open the PR, then merge into `features/tiliakoos` and rebuild.
- **Only for Nick** (personal defaults, docs, local hacks): commit straight to `features/tiliakoos` and rebuild.

When unsure, treat it as upstreamable. Starting on a topic branch and later deciding not to send it costs nothing; the reverse means untangling it from personal commits.

Never build a PR diff on top of personal-branch commits. Never commit to `main`.

Full procedure, including what to do while a PR is under review and after it merges, is in the vault's `fork-workflow.md`.

## Non-negotiables

1. **`npm install` needs `--force`** on Apple Silicon (`@esbuild/darwin-x64` is a hard dependency; upstream PR #635 is the real fix and is still open). Afterwards run `git checkout -- package-lock.json`. Never commit that lockfile churn.
2. **`npm run build:native` rewrites `src/native/soulver-calculator/Package.resolved`.** Revert it before committing. It must never appear in a PR diff.
3. **Re-sign after packaging.** `codesign --force --deep --sign - out/mac-arm64/SuperCmd.app`. Without it the bundle fails `codesign --verify` and macOS can kill it at launch.
4. **Quit the running app before replacing `/Applications/SuperCmd.app`,** and expect Accessibility and Input Monitoring to need re-granting afterwards.
5. **Never commit credentials or a real Raycast backup.** Tests build their own fixtures.

## Before opening a PR

Upstream CONTRIBUTING asks for: a `feat/` or `fix/` prefix, `npm run build` passing, changes tested with `npm run dev`, one concern per PR, and a description covering what changed, why, compatibility impact, and how it was tested. `npm test` must pass. Confirm `package-lock.json` and `Package.resolved` are absent from the diff.

Full commands in the vault's `fork-workflow.md`.
