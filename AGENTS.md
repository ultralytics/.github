# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, etc.) when working with code in this repository. CLAUDE.md is a symlink to this file.

This repository (AGPL-3.0) is GitHub's special `.github` repository for the Ultralytics org. It holds the org profile page and the default community-health files — issue and pull request templates, code of conduct, security policy, and funding — that every Ultralytics repository without its own copy inherits.

## Core Principles (CRITICAL)

**Less is more. The simplest solution is the best solution.** The action hierarchy for every change: **Delete > Replace > Add**.

1. **Solve at the owner**: Put behavior in the code path that owns or observes it. For fixes, never guard a symptom with a staleness check, initialization flag, skip-first-call branch, or `try/except` around broken logic; relocate the trigger and delete the wrong path. For features, extend the existing owner rather than creating a parallel abstraction.
2. **Search and reuse first**: Search the whole repository before creating a feature, component, helper, workflow, or utility. Reuse or adapt what exists, consolidate in-scope duplication in the shared owner, and delete duplicate paths. Three similar lines beat a helper nobody else calls.
3. **Delete and modify existing code before creating new code**: Bugfixes are net-negative by default unless deletion and relocation are demonstrably impossible. A new file must first prove it cannot fit cleanly in an existing owner.
4. **Keep scope minimal**: Implement only the simplest complete solution. Avoid impossible-state handling, speculative flags, compatibility shims, policy scaffolding, and unrelated cleanup. Tests are out of scope by default — rely on existing coverage and focused validation; only an uncovered, high-risk regression path justifies minimal new test code.
5. **Ship zero-regression, production-ready changes**: Understand what you remove instead of retaining broken code as insurance. Remove unused imports, functions, types, files, and comments; run relevant cleanup checks; and thoroughly debug and validate the changed owner. Do not break existing features or workflows unless the PR intentionally removes them with evidence.

**Review gate:** for every addition, the reviewer decides whether deleting or changing existing code would have fixed the problem instead — if it would, that is a blocking finding. A missing or thin PR description is never itself a finding.

NEVER push to `main`. NEVER force push. Always start work in a new git worktree (`git worktree add`) on a feature branch and open a PR — never edit the primary checkout directly, it may hold in-flight work.

## PR Workflow

After opening a PR:

1. Wait for the automated PR review and auto-format commit from Ultralytics Actions (`format.yml`), then pull and address every finding.
2. Review the full diff in-session against the Core Principles, performance, and the review gate above, then batch the fixes into one commit and push. After each round of bot or human commits, pull and resume the same reviewer on `<last-reviewed-sha>..HEAD` plus anything that delta could have invalidated. Repeat until the local head matches the live head.
3. Hand off or merge only on a clean final pass: one cold full-diff review returning LGTM with no findings, on a head that is still live at merge time.
4. Never fight other commits: Ultralytics Actions pushes auto-format and header commits, and multiple users may work on the same PR. `git pull --rebase` before pushing; never reset or revert commits you did not author.
5. After the PR merges, clean up: remove local worktrees and branches for it, then `git checkout main && git pull`.

## Commands

This repo ships config, templates, and docs — not code. There is no package, no build, no test suite, and no local lint step to run.

- Formatting and spelling are applied in CI, not locally: `.github/workflows/format.yml` (Ultralytics Actions) runs prettier over YAML/JSON/Markdown, codespell over spelling, and pushes a fixup commit to the PR branch on open/synchronize. `git pull --rebase` after it runs.
- Issue-template forms and the org profile only render on GitHub — there is no local preview command; open a PR to see them.

## Architecture

This is GitHub's special [`.github` repository](https://docs.github.com/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file) for the `ultralytics` org. It contains no source code — only the org profile, default community-health files, issue/PR templates, and CI config. The community-health files and templates are org-wide defaults: any org repo that does not define its own copy inherits them, so editing them can affect every repository in the org. The workflows and `profile/README.md` apply to this repo only.

- `profile/README.md` renders as the public org landing page at github.com/ultralytics.
- `.github/ISSUE_TEMPLATE/` holds the default issue forms (`bug-report.yml`, `feature-request.yml`, `question.yml`) plus `config.yml`, which sets `blank_issues_enabled: true` and the docs/forum/Discord/discussions contact links.
- `PULL_REQUEST_TEMPLATE.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, and `FUNDING.yml` are default community-health files inherited by org repos lacking their own. `LICENSE` (AGPL-3.0) and `README.md` are this repo's own and are not inherited.
- The workflows are `.github/workflows/format.yml` (Ultralytics Actions on issue/PR events: prettier for YAML/JSON/Markdown/CSS, codespell, Lychee link checks, and AI-generated labels/summaries via `OPENAI_API_KEY`) and `.github/workflows/cla.yml` (Ultralytics CLA check on PRs). CodeQL also runs on pushes and PRs through the org's default code-scanning setup (no workflow file here), so PRs show Actions, CLA, and CodeQL checks.

## Conventions

- License headers: every YAML file opens with `# Ultralytics 🚀 AGPL-3.0 License - https://ultralytics.com/license`. Ultralytics Actions adds and maintains these — don't add or revert them manually. Markdown files (README, health docs, this file) carry no header.
- Issue templates are GitHub [issue forms](https://docs.github.com/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-issue-forms) (YAML schema), not classic Markdown templates; edit the `body:` fields, keep valid form syntax.
- Formatting is bot-owned: when Ultralytics Actions reformats or fixes spelling it pushes a commit to the PR branch (Markdown/YAML-only PRs often need none), so `git pull --rebase` after its run; its prettier output can differ from a local prettier run.
- No versioning or release process — nothing is published from this repo; changes take effect the moment they merge to `main`.
