---
name: skills-downloader
description: >-
  Browse, search, and install AI agent skills from the REA Agent Skills Hub.
  Use when users want to discover available skills, install a skill, remove a
  skill, update a skill, or check what skills are installed. Triggers on
  requests like "install a skill", "download a skill", "list available skills",
  "what skills are available", "remove skill X", or "show installed skills".
---

# Skills Downloader

## Critical Rules

- You MUST fetch skills.json first to find the correct repo URL. Never guess repo URLs.
- Skills are markdown files and folders cloned from git repos. Never run `npm install`, `pip install`, or any package manager.
- Always remove `.git/` from the installed skill directory after copying.
- Never use HTTPS git URLs — they prompt for passwords. Use SSH (`git@git.realestate.com.au:`).
- Always ask the user where to install before cloning. Use the ask question tool to do so.
- Only install into: `.github/skills/`, `.claude/skills/`, `~/.claude/skills/`, or `~/.copilot/skills/`.

## How to Install a Skill

### 1. Fetch the skill catalog to find the correct repo

This step is mandatory. Run:

```bash
gh api --hostname git.realestate.com.au /repos/devlob/agent-skills-hub/contents/website/public/skills.json --jq '.content' | base64 -d
```

If `gh` is not installed: `brew install gh`. If not authenticated: `gh auth login --hostname git.realestate.com.au`.

The JSON contains `{ "skills": [...] }`. Find the skill by `name`. Each skill has:
- `repo` — the source repository (e.g. `keegan-street/construct-kit-skill`). Use this for cloning.
- `skill_path` — path within the repo (e.g. `.github/skills/adr-coauthoring` or `.` for whole repo). Use this for copying.

### 2. Ask the user where to install

Before cloning, ask the user to choose:
1. `.github/skills/<skill-name>/` — Project-level, recommended. Works for both Claude Code and GitHub Copilot.
2. `.claude/skills/<skill-name>/` — Project-level, Claude Code only.
3. `~/.claude/skills/<skill-name>/` — User-level for Claude Code. Available across all projects.
4. `~/.copilot/skills/<skill-name>/` — User-level for GitHub Copilot. Available across all projects.

Wait for the user to respond.

### 3. Clone and copy

Use the `repo` and `skill_path` values from the catalog (step 1). Run:

```bash
TMPDIR=$(mktemp -d)
git clone --depth 1 git@git.realestate.com.au:<REPO>.git "$TMPDIR/repo"
mkdir -p <INSTALL_PATH>
cp -r "$TMPDIR/repo/<SKILL_PATH>/." "<INSTALL_PATH>/"
rm -rf "<INSTALL_PATH>/.git"
rm -rf "$TMPDIR"
ls -la "<INSTALL_PATH>/"
```

If SSH fails, fall back to: `gh repo clone <REPO> "$TMPDIR/repo" --hostname git.realestate.com.au -- --depth 1`

### 4. Confirm

Tell the user the skill is installed, its location, and how to use it (`/<skill-name>` for Claude Code, auto-activates for Copilot).

## List Available Skills

Fetch the catalog (step 1 above), group by `name`, and show a table with skill name, description, and number of repos.

## Remove a Skill

Check `.github/skills/<name>/`, `.claude/skills/<name>/`, `~/.claude/skills/<name>/`, `~/.copilot/skills/<name>/`. Ask the user which to remove, confirm, then `rm -rf`.

## Show Installed Skills

Run `ls` on all four skill directories and show what is installed in each.

## Errors

| Error | Fix |
|-------|-----|
| `gh` not installed | `brew install gh` |
| `gh` not authenticated | `gh auth login --hostname git.realestate.com.au` |
| SSH clone fails | Fall back to `gh repo clone`. Test with `ssh -T git@git.realestate.com.au` |
| Repo not found | Catalog updates every 4 hours — repo may have moved |
