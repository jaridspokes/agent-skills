# Documentation Gap Analyzer

Analyzes a codebase to identify undocumented or under-documented areas, cross-references findings against REA's internal knowledge base (Glean) to avoid suggesting documentation that already exists, and produces a prioritized documentation gap report as a GitHub issue with suggested topics, outlines, and Mermaid diagram plans.

## Features

- 🔍 **Deep Codebase Scanning** - Analyzes code structure, configs, infrastructure, APIs, data models, and tests
- 🌐 **Glean Cross-Reference** - Searches REA's internal knowledge base to avoid suggesting docs that already exist
- 📊 **Prioritized Gap Report** - Ranks documentation gaps by impact on onboarding and productivity (P0-P3)
- 📐 **Mermaid Diagram Plans** - Suggests specific diagram types for each documentation topic
- 📋 **GitHub Issue Output** - Creates a structured GitHub issue with full analysis and suggested table of contents

## Quick Start

### Prerequisites
- Claude Code installed and configured via Omnia
- GitHub CLI (`gh`) authenticated with the target repository
- Glean MCP configured (see `.vscode/mcp.json`)

### Installation

1. Find the skill in the REA agent skills hub:
   https://pages.git.realestate.com.au/devlob/agent-skills-hub/
   Search for `docs-gap-analyzer`

2. Open your Claude Code terminal and run:
   ```
   use skills-downloader to install the docs-gap-analyzer skill
   ```

### Usage

1. Navigate to the repository you want to analyze.
2. Trigger the skill:
   ```
   /docs-gap-analyzer
   ```
3. The agent will scan the codebase, cross-reference with Glean, and present a summary.
4. Confirm to create the GitHub issue with the full gap analysis.

## Output

The skill creates a GitHub issue titled `[Docs] Documentation Gap Analysis: {repo-name}` containing:
- Executive summary with gap counts by priority
- Existing documentation inventory (both in-repo and external via Glean)
- Detailed gap descriptions with suggested outlines and Mermaid diagram plans
- Suggested `docs/` folder structure (table of contents)
- Recommended implementation order

## Pairing with docs-generator

Use this skill first to identify gaps, then use the `docs-generator` skill to actually create the documentation from the generated issue. The GitHub issue acts as the contract between the two skills.
