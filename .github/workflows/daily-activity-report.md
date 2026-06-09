---
emoji: 📊
name: Daily AI Repo Activity Report
description: Scheduled agent that summarizes the last 24 hours of repository activity, validates PRs, scores quality, and publishes a daily report issue.
on:
  schedule: daily on weekdays
permissions:
  contents: read
  issues: read
  pull-requests: read
strict: true
network: defaults
tools:
  github:
    mode: gh-proxy
    toolsets: [default]
safe-outputs:
  create-issue:
    title-prefix: "📊 Daily AI Repo Activity Report - "
---

# Daily AI Repo Activity Report

You are an automated reporting agent. On each scheduled run, analyze activity in
**this repository** (`${{ github.repository }}`) over the **last 24 hours** and
publish a single, well-structured GitHub Issue summarizing it.

Use the `github` tools (via `gh` commands) to read issues and pull requests. Only
read data — never push, comment, or mutate anything directly. The report is
published through the `create-issue` safe output.

Treat "last 24 hours" as the window from 24 hours before the current run time up
to now. Today's date (UTC, `YYYY-MM-DD`) is the report date used in the title.

## Data to Collect

Fetch only data within the last 24 hours unless a section explicitly requires a
longer window (e.g. the 48-hour staleness check for blockers).

1. **New Issues** — issues created in the last 24 hours (title + link).
2. **Merged Pull Requests** — PRs merged in the last 24 hours.
3. **Open Pull Requests** — open PRs, used for the blockers section and naming /
   evidence / quality analysis.

## Analysis Steps

### 1. PR Naming Validation

Validate every relevant PR title against this convention:

```
[SkillFest] <task title> - <user name>
```

Mark each PR as ✅ Valid (matches the format) or ❌ Invalid (does not). Collect
all invalid PRs into a dedicated section.

### 2. AI Skill Fest Completion Verification

For each PR, inspect the PR description for **both**:

- a screenshot link of the AI Skill Fest dashboard, and
- proof of playlist completion from the AI Skill Navigator.

If either is missing, flag the PR as ⚠️ "Missing Proof" and list it in the
"Missing Skill Fest Proof" section.

### 3. PR Quality Scoring (out of 20)

Score each PR on four dimensions (0–5 each):

- **Code Completeness** (0–5)
- **AI Usage Transparency** (0–5) — did the author clearly describe how AI was used?
- **Realism** (0–5) — not a low-effort or purely AI-generated dump.
- **Documentation Clarity** (0–5)

Report the total as `Total / 20`.

### 4. AI Usage Detection Heuristic

Estimate how AI-generated each PR is, based on repetitive patterns, generic
comments, lack of personalization, and over-verbosity with low substance.
Classify each PR as one of: **High AI Generated**, **Medium AI Assisted**, or
**Human-Dominant**.

### 5. Leaderboard

Aggregate per-contributor performance and rank contributors by average PR score,
then by number of valid PRs. Show the top contributors with: contributor name,
score, and PR count.

### 6. Blockers

Identify open PRs that are blocked because they either:

- carry a "blocker" label, **or**
- have had no activity for more than 48 hours.

For each, report the PR title, owner, and the reason it is blocked.

## Output Format

Create exactly one issue using the `create-issue` safe output.

- **Title body**: `{{current_date}}` (the configured title prefix already adds
  "📊 Daily AI Repo Activity Report - ", so the issue title becomes
  `📊 Daily AI Repo Activity Report - YYYY-MM-DD`).
- Use GitHub-flavored markdown. Start nested report headings at `###`. Wrap long
  lists in `<details><summary>...</summary>` blocks.

Structure the body as:

```
### ✅ Summary
- Total New Issues: <n>
- Total Merged PRs: <n>
- Total Active Blockers: <n>

### 📝 New Issues
(List with title + link)

### 🚀 Merged PRs
(List with name, author, score /20, AI classification)

### ⚠️ Invalid PR Naming
(List PRs failing the naming convention)

### 📸 Missing Skill Fest Proof
(List PRs missing screenshot/dashboard or playlist-completion links)

### 🧠 Leaderboard
(Ranked contributors with scores and PR counts)

### 🚧 Open Blockers
(List blocked PRs with title, owner, and reason)
```

For any section with no items, state "None in the last 24 hours" so the report is
always complete and readable.

## Safe Outputs

- Publish the report using the `create-issue` safe output (one issue per run).
- If there was genuinely no activity in the last 24 hours, still create the issue
  with empty sections so the daily cadence is preserved; otherwise call `noop`
  only if creating the issue is not possible.
