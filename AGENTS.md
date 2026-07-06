# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, etc.) when working with code in this repository. CLAUDE.md is a symlink to this file.

## Core Principles (CRITICAL)

**Delete > Replace > Add.** Before writing any change, answer in order: what can I delete? what can I replace? only then, what must I add?

The most common agent failure in this repo is reaching for the locally-safest edit — a new guard, flag, or helper — instead of fixing ownership. These tripwires override that instinct:

1. **Never guard a symptom — relocate the trigger.** A fix that adds a condition to suppress bad behavior (a staleness check, an is-initialized flag, a skip-first-call guard, a try/except around broken logic) is wrong by default. Find the code path that should own the behavior, move the logic there, and delete the code that got it wrong. Example: a warning fired from stale state; the right fix was not a recency guard — it deleted the stale detection and moved the trigger into the code path that observes the event live.
2. **Bugfixes are net-negative by default.** A bugfix that adds more lines than it removes needs a one-sentence justification in the PR body naming why deletion and relocation were impossible.
3. **Search the repo before creating anything.** Before adding a template, workflow, or health file, search the whole repo — it likely exists (this repo holds no application code; its reusable assets are the community-health files under `.github/` and the repo root that every other ultralytics repo inherits). If two templates or workflows grow the same logic, consolidate into one and delete the duplicate. Avoid premature abstraction — three similar lines beat a helper nobody else calls.
4. **Deletion beats caution.** Zero regression means understanding the code you remove, not leaving it in place as insurance. Keeping broken or duplicated code "to be safe" is itself the regression: it is how repos rot. All changes must still ship debugged, validated, and production ready.

**Output gate:** every PR body must contain a `Deleted:` line naming the code removed (functions, branches, files, config). Features must name what they reused or consolidated. `Deleted: nothing` demands the rule-2 justification.

**Review gate:** adversarial reviewers must answer two questions before LGTM: (a) what could have been deleted instead of added? (b) does any added condition suppress a symptom rather than relocate a trigger? A finding on either blocks LGTM.

**This file is code — additions require deletions.** To add a rule here, remove or merge one. When everything is emphasized, nothing is.

**NEVER push to `main`. NEVER force push.** Always start work in a new git worktree (`git worktree add`) on a feature branch and open a PR — never edit the primary checkout directly, it may hold in-flight work.

**Files here are org-wide defaults.** Any ultralytics repo without its own copy inherits these community-health files, so a change here can affect every repository in the org — verify the blast radius before editing.

## PR Workflow

After opening a PR:

1. Wait for the automated PR review and auto-format commit from Ultralytics Actions (`format.yml`), then pull and address every finding.
2. Launch an independent adversarial review agent with cold context (just the PR diff and this file) to hunt for bugs, regressions, and Core Principles violations — use the Codex CLI, one fresh `codex exec` run per round. Fix, push, and repeat until a fresh run reports LGTM.
3. Never fight other commits: Ultralytics Actions pushes auto-format and header commits, and multiple users may work on the same PR. `git pull --rebase` before pushing; never force-push, reset, or revert commits you did not author.
4. After the PR merges, clean up: remove local worktrees and branches for it, then `git checkout main && git pull`.

## Commands

This repo ships config, templates, and docs — not code. There is no package, no build, no test suite, and no local lint step to run.

- Formatting and spelling are applied in CI, not locally: `.github/workflows/format.yml` (Ultralytics Actions) runs prettier over YAML/JSON/Markdown, codespell over spelling, and pushes a fixup commit to the PR branch on open/synchronize. `git pull --rebase` after it runs.
- Issue-template forms and the org profile only render on GitHub — there is no local preview command; open a PR to see them.

## Architecture

This is GitHub's special [`.github` repository](https://docs.github.com/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file) for the `ultralytics` org. It contains no source code — only default community-health files, the org profile, issue/PR templates, and CI. Any org repo that does not define its own copy inherits these files, so edits here propagate org-wide.

- `profile/README.md` renders as the public org landing page at github.com/ultralytics.
- `.github/ISSUE_TEMPLATE/` holds the default issue forms (`bug-report.yml`, `feature-request.yml`, `question.yml`) plus `config.yml`, which sets `blank_issues_enabled: true` and the docs/forum/Discord/discussions contact links.
- `PULL_REQUEST_TEMPLATE.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, and `FUNDING.yml` are default community-health files inherited by org repos lacking their own. `LICENSE` (AGPL-3.0) and `README.md` are this repo's own and are not inherited.
- Active CI is `.github/workflows/format.yml` only: Ultralytics Actions on issue/PR events (Ruff+docformatter for Python, prettier for YAML/JSON/Markdown/CSS, codespell, and AI-generated labels/summaries via `OPENAI_API_KEY`). CodeQL runs through the org's default code-scanning setup, not a workflow file in this repo.
- `workflows/stale.yml` and the root `dependabot.yml` are inert at their current paths: GitHub reads workflows only from `.github/workflows/` and Dependabot config only from `.github/dependabot.yml`. Both are legacy from the YOLOv5 template — moving them into `.github/` would activate them, so treat that as an intentional behavior change, not a cleanup.

## Conventions

- License headers: every YAML file opens with `# Ultralytics 🚀 AGPL-3.0 License - https://ultralytics.com/license`. Ultralytics Actions adds and maintains these — don't add or revert them manually. Markdown files (README, health docs, this file) carry no header.
- Issue templates are GitHub [issue forms](https://docs.github.com/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-issue-forms) (YAML schema), not classic Markdown templates; edit the `body:` fields, keep valid form syntax.
- Formatting is bot-owned: expect an Ultralytics Actions commit on every PR branch, and note its prettier output can differ from a local prettier run.
- No versioning or release process — nothing is published from this repo; changes take effect the moment they merge to `main`.
