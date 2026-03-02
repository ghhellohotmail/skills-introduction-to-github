# CLAUDE.md — AI Assistant Guide

This file provides context for AI assistants (Claude, Copilot, etc.) working in this repository.

## What This Repository Is

This is a **GitHub Skills interactive course template** — [Introduction to GitHub](https://github.com/skills/introduction-to-github). It is not a traditional application codebase. It contains:

- **GitHub Actions workflows** that automatically guide learners through steps
- **Markdown content** describing each course step
- **Images** for visual step-by-step guidance
- **No build system, no package manager, no application code**

The repository is designed to be used as a **GitHub template** that learners fork/clone. Once forked, GitHub Actions detect learner actions (creating branches, committing files, opening/merging PRs) and automatically advance the learner through the course.

## Repository Structure

```
.
├── .github/
│   ├── steps/                    # Step content displayed to learners
│   │   ├── -step.txt             # Current step counter (values: 0–4, X)
│   │   ├── 0-welcome.md
│   │   ├── 1-create-a-branch.md
│   │   ├── 2-commit-a-file.md
│   │   ├── 3-open-a-pull-request.md
│   │   ├── 4-merge-your-pull-request.md
│   │   └── X-finish.md
│   ├── workflows/                # GitHub Actions (one per course step)
│   │   ├── 0-welcome.yml
│   │   ├── 1-create-a-branch.yml
│   │   ├── 2-commit-a-file.yml
│   │   ├── 3-open-a-pull-request.yml
│   │   └── 4-merge-your-pull-request.yml
│   └── dependabot.yml            # Monthly updates for GitHub Actions versions
├── images/                       # PNG screenshots referenced in step markdown files
├── .gitignore                    # Standard binary/OS/log ignores
├── LICENSE                       # MIT License (GitHub, Inc.)
├── PROFILE.md                    # Example file learners create during the course
└── README.md                     # Course landing page (updated dynamically by Actions)
```

## Course Progression

The course progresses through numbered steps. The current step is tracked in `.github/steps/-step.txt`.

| Step | Value | Trigger Event | Learner Action |
|------|-------|---------------|----------------|
| Welcome | 0 → 1 | Push to `main` | Repository set up |
| Create a branch | 1 → 2 | Branch `my-first-branch` created | Create branch named exactly `my-first-branch` |
| Commit a file | 2 → 3 | Push to `my-first-branch` | Commit `PROFILE.md` to the branch |
| Open a pull request | 3 → 4 | PR opened/reopened from `my-first-branch` | Open PR targeting `main` |
| Merge pull request | 4 → X | PR merged into `main` | Merge the PR |
| Finish | X | — | Course complete |

Step transitions are handled by the `skills/action-update-step@v2` action, which updates both `README.md` and `.github/steps/-step.txt`.

## GitHub Actions Workflow Conventions

All workflow files follow a consistent two-job pattern:

```yaml
jobs:
  get_current_step:          # Job 1: reads -step.txt and exposes it as output
    steps:
      - uses: actions/checkout@v4
      - id: get_step
        run: echo "current_step=$(cat ./.github/steps/-step.txt)" >> $GITHUB_OUTPUT
    outputs:
      current_step: ${{ steps.get_step.outputs.current_step }}

  on_<action>:               # Job 2: runs only when step number matches
    needs: get_current_step
    if: >-
      ${{ !github.event.repository.is_template
          && needs.get_current_step.outputs.current_step == <N>
          && <additional event conditions> }}
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: skills/action-update-step@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          from_step: <N>
          to_step: <N+1>
```

**Key rules:**
- The `!github.event.repository.is_template` guard prevents workflows from running on the original template repository itself — only on learner forks.
- All workflows require `contents: write` permission to update step metadata.
- `fetch-depth: 0` is always used to get full branch history.
- Workflow file names mirror step numbers: `{N}-{description}.yml`.

## Markdown Conventions

- Step files live in `.github/steps/` and are named `{N}-{description}.md`
- Author notes are embedded as HTML comments: `<!-- <<< Author notes: ... >>> -->`
- Images are referenced with absolute paths from repo root: `/images/filename.png`
- External links always point to `docs.github.com` for GitHub documentation
- Activities are marked with `:keyboard: Activity:` heading
- Each step file contains exactly one `## Step N: Title` heading

## What AI Assistants Should Know

**This repo has no:**
- Build commands (`npm run build`, `make`, etc.)
- Test suite
- Linter configuration
- Dependencies or package manager files

**When modifying workflows:**
- Keep the two-job pattern (step check + conditional action job)
- Always guard with `!github.event.repository.is_template`
- Match step numbers in file names, conditions, and `from_step`/`to_step` values
- Use `actions/checkout@v4` and `skills/action-update-step@v2` (check dependabot.yml for latest pinned versions)

**When modifying step content (`.github/steps/*.md`):**
- Maintain the `## Step N: Title` heading format
- Keep `:keyboard: Activity:` sections with numbered lists
- Reference images from `/images/` directory
- Link to official GitHub docs for concept explanations

**When modifying README.md:**
- The file is dynamically rewritten by GitHub Actions — the `<header>` and `<footer>` HTML tags delimit sections
- The body between tags is replaced by the current step's markdown content
- Do not restructure the header/footer delimiters

## Git Workflow for This Repository

The primary branches are:
- `master` (or `main`) — canonical course state
- `my-first-branch` — the branch learners create during the course (specific name required)

For AI-assisted development work, use the `claude/` prefixed branch specified in session context.

**Branch naming:** `claude/<session-id>` for AI development branches.

**Commit style:** Short imperative messages matching the existing log style:
- `Update to X in STEP and README.md`
- `Add PROFILE.md`
- `Create PROFILE.md`

## Current State

- **Course status**: Completed (step X — `.github/steps/-step.txt` contains `X`)
- **Last learner action**: Merged PR #1 from `my-first-branch` → `master`
- **Dependabot**: Monitors GitHub Actions dependencies monthly
