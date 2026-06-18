---
name: code-review
description: >-
  Automated code review for pull requests using multiple specialized agents with
  confidence-based scoring. Use when the user asks to run /code-review,
  review a pull request, audit CLAUDE.md compliance, or find high-signal bugs
  in a PR diff. Supports optional --comment mode to post review results.
---

# Code Review Skill

Provide a code review for the current pull request.

Usage:
- `/code-review`
- `/code-review --comment`

Behavior:
- If `--comment` is omitted: output findings in the terminal/chat only.
- If `--comment` is provided: post review output to the PR as comments.

Agent assumptions (applies to all agents and subagents):
- All tools are functional and will work without error. Do not test tools or make exploratory calls.
- Only call a tool if it is required to complete the task.

Follow these steps precisely:

1. Launch a lightweight subagent to check if any condition below is true:
- The pull request is closed.
- The pull request is a draft.
- The pull request does not need code review (automated PR or trivial change that is obviously correct).
- Claude has already commented on this PR (check `gh pr view <PR> --comments` for comments left by `claude`).

If any condition is true, stop and do not proceed.

Note: still review Claude-generated PRs.

2. Launch a lightweight subagent to return file paths (not contents) for relevant `CLAUDE.md` files, including:
- Root `CLAUDE.md`, if present.
- Any `CLAUDE.md` files in directories containing files modified by the PR.

3. Launch a stronger subagent to summarize pull request changes.

4. Launch 4 review subagents in parallel. Each returns a list of issues with a description and reason (`CLAUDE.md adherence` or `bug`).
- Agents 1 and 2: CLAUDE.md compliance reviewers.
- Agent 3: bug reviewer focusing only on changed lines.
- Agent 4: bug reviewer focusing on introduced code in changed lines.

CLAUDE.md scope rule:
- For any file, only consider `CLAUDE.md` files in matching or parent paths.

High-signal rule:
- Only flag issues that are clearly real and important.
- Include syntax/type/import failures, clear logic errors, and unambiguous CLAUDE.md rule violations.
- Do not flag style nits, subjective improvements, or input-dependent speculation.

5. For each issue found by bug/compliance reviewers, launch validation subagents in parallel:
- Validate that each issue is truly real with high confidence.
- For CLAUDE.md issues, validate both scope and exact rule violation.

6. Filter out all unvalidated issues.

7. Output review summary:
- If issues found: list each issue briefly.
- If none found: `No issues found. Checked for bugs and CLAUDE.md compliance.`

If `--comment` was not provided, stop.

If `--comment` was provided and no issues were found, post:

---
## Code review

No issues found. Checked for bugs and CLAUDE.md compliance.
---

If `--comment` was provided and issues were found, continue.

8. Draft a private list of comments to leave (do not post this draft list).

9. Post inline comments for each issue. For each comment:
- Provide a brief issue description.
- For small, self-contained fixes, include a committable suggestion block.
- For larger fixes (6+ lines, structural changes, or multi-location changes), describe suggested fix without a suggestion block.
- Never provide a committable suggestion unless it fully fixes the issue by itself.
- Post only one comment per unique issue.

False positives to always filter out:
- Pre-existing issues.
- Things that look wrong but are correct.
- Pedantic nitpicks.
- Issues that a linter would catch.
- General quality concerns unless explicitly required in CLAUDE.md.
- Issues explicitly silenced in code (for example via lint-ignore comments).

Implementation notes:
- Use `gh` CLI for PR metadata, diffs, and top-level comments.
- Include links/citations for every flagged issue.
- Use full-SHA GitHub links in this exact shape:
  `https://github.com/owner/repo/blob/<full-sha>/path/file.ext#L<start>-L<end>`
- Include at least one line of context before and after the key line(s) when possible.
