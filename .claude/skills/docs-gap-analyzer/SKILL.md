---
name: docs-gap-analyzer
description: Analyzes a codebase to identify undocumented or under-documented areas, cross-references findings against REA's internal knowledge base via Glean, and produces a prioritized documentation gap report with suggested topics, Mermaid diagram plans, and a table of contents. Creates a GitHub issue with the full analysis.
owner:
  team: PropTrack CPD
  slack: "#pt-property-data-support"
categories:
  - documentation
  - analysis
tags:
  - documentation
  - gap-analysis
  - github
  - mermaid
maturity: beta
version: 1.0.0
---

# Documentation Gap Analyzer

This skill guides the agent to act as a senior Technical Writer and Documentation Strategist. Your task is to perform a deep, systematic analysis of a codebase to identify what is NOT documented, cross-reference your findings against REA's internal knowledge base (Glean) to avoid duplicating existing documentation, and produce a comprehensive documentation gap report as a GitHub issue.

## Core Principles

1. **Exhaustive Codebase Scanning:** Analyze every significant structural element — do not limit yourself to what is obvious. Look at code structure, configuration, infrastructure, tests, and implicit architectural decisions.
2. **Cross-Reference Before Recommending:** ALWAYS search Glean before suggesting a documentation topic. If documentation exists externally (Confluence, Notion, etc.), note it as "Covered Externally" with a link, not as a gap.
3. **Prioritize by Impact:** Rank gaps by how much they would hinder a new team member's ability to onboard, debug, or contribute. Onboarding and architecture docs are higher priority than niche configuration docs.
4. **Diagram-First Thinking:** For every documentation topic, consider which Mermaid diagram type would best visualize the concept. Not all topics need diagrams, but most architecture and flow topics do.
5. **Actionable Output:** The GitHub issue must be structured so that each suggested topic could become an independent work item. Each topic should have a clear scope, suggested outline, and diagram plan.

## Workflow

### Phase 1: Codebase Discovery and Structural Analysis

This is the most critical phase. You must build a comprehensive mental model of the entire codebase before identifying what is missing.

#### Step 1.1: Identify the Repository

- Confirm the current working directory is the target repository.
- Run `git remote -v` to identify the repo name and GitHub URL.
- Run `git log --oneline -20` to understand recent development activity.
- Check for an existing `docs/` folder and read any content within it.

#### Step 1.2: Scan Codebase Structure

Systematically scan each of these structural categories:

| Category | What to Scan | Approach |
|----------|-------------|----------|
| **Project Root** | README.md, CONTRIBUTING.md, CHANGELOG.md, LICENSE, Makefile, docker-compose.yml, Dockerfile | Read each file if it exists |
| **Source Code** | Entry points, module boundaries, public APIs, exported functions/classes | Glob for `src/**/*.{ts,js,py,scala,java}`, identify main/index files |
| **Configuration** | Environment variables, config files, feature flags, secrets management | Glob for `*.{yml,yaml,json,toml,ini,env.example}`, `.env.example` |
| **Infrastructure** | IaC files (Terraform, CDK, CloudFormation), CI/CD pipelines | Glob for `**/*.tf`, `**/cdk.json`, `.github/workflows/*`, `buildkite/*`, `Jenkinsfile` |
| **Data Models** | Database schemas, migrations, ORM models, Protobuf/Avro schemas | Glob for `**/migrations/**`, `**/models/**`, `**/*.proto`, `**/*.avsc` |
| **API Definitions** | OpenAPI specs, GraphQL schemas, route definitions | Glob for `**/swagger.*`, `**/openapi.*`, `**/*.graphql`, route files |
| **Tests** | Test structure, coverage reports, test fixtures | Glob for `**/test*/**`, `**/*spec*`, `**/*test*` |
| **Dependencies** | Package manifests, dependency lock files | Read `package.json`, `build.sbt`, `pom.xml`, `requirements.txt`, `pyproject.toml`, `Cargo.toml` |

#### Step 1.3: Identify Architectural Patterns

Based on the scan, determine:
- What language(s) and framework(s) are used?
- Is this a monolith, microservice, library, data pipeline, or CLI tool?
- What are the major modules/packages and their relationships?
- What external services/APIs does this interact with?
- What is the deployment target (AWS Lambda, ECS, Kubernetes, etc.)?
- What data stores are used (RDS, DynamoDB, S3, Kafka, etc.)?

#### Step 1.4: Catalog Existing Documentation

Build an inventory of what IS documented:
- Root README.md — what sections does it cover?
- Any `docs/` folder content
- Inline code comments and docstrings (sample 5-10 key files)
- Any ADRs (Architecture Decision Records) in `docs/adr/` or `adr/`
- Any generated docs (Javadoc, Scaladoc, TypeDoc, Sphinx)
- `CONTRIBUTING.md`, `CHANGELOG.md`, `RUNBOOK.md`

### Phase 2: Glean Cross-Reference and External Documentation Audit

#### Step 2.1: Search Glean for Existing External Documentation

Using the Glean MCP `search()` function, perform targeted searches for each of these categories. Use the repository name, team name, and key domain terms as search keywords.

Searches to perform (adapt keywords based on Phase 1 findings):
1. `"{repo-name} documentation"` — general docs
2. `"{repo-name} architecture"` — architecture docs
3. `"{repo-name} onboarding"` — onboarding guides
4. `"{repo-name} runbook"` or `"{repo-name} playbook"` — operational docs
5. `"{repo-name} API"` — API documentation
6. `"{repo-name} deployment"` — deployment guides
7. `"CPD {domain-term}"` for each key domain term identified in Phase 1
8. `"{repo-name} ADR"` — architecture decision records

#### Step 2.2: Read Top Results

For each search, use `read_document()` on the top 2-3 results to determine:
- Is this document still current and accurate?
- What does it cover vs. what does it miss?
- Is it findable (i.e., would a new team member discover it)?

#### Step 2.3: Build External Documentation Map

Create an internal mapping:
```
Topic -> [External Doc Title, URL, Coverage Status (Full/Partial/Outdated/None)]
```

### Phase 3: Gap Analysis and Report Generation

#### Step 3.1: Documentation Category Taxonomy

Use this standard taxonomy. Apply all that are relevant, skip those that genuinely do not apply:

| Category | Icon | Description |
|----------|------|-------------|
| Architecture Overview | 🏗️ | System design, component relationships, technology choices |
| Data Models & Schema | 🗃️ | Database design, entity relationships, data dictionary |
| API Reference | 🔌 | Endpoints, request/response schemas, auth, error codes |
| Data Flow & Pipelines | 🔄 | How data moves through the system, transformations, ETL/ELT |
| Configuration & Environment | ⚙️ | Env vars, config files, feature flags, secrets |
| Deployment & Infrastructure | 🚀 | How to deploy, IaC, environments, CI/CD |
| Testing Strategy | 🧪 | How to run tests, what's covered, testing philosophy |
| Onboarding & Getting Started | 📖 | Local setup, prerequisites, first contribution guide |
| Operational Runbook | 🔧 | Incident response, common issues, monitoring, alerting |
| Security & Access Control | 🔒 | Auth model, permissions, data sensitivity, compliance |
| Domain Concepts | 📚 | Business logic explanations, glossary, domain model |
| Decision Log (ADRs) | 📝 | Why certain technical decisions were made |

#### Step 3.2: Gap Detail Requirements

For each identified documentation gap, produce:

1. **Topic Title**: Clear, specific name (e.g., "Data Pipeline Architecture: Property Ingestion Flow")
2. **Category**: From the taxonomy above
3. **Priority**: `P0 Critical` / `P1 High` / `P2 Medium` / `P3 Low`
   - P0: Missing and blocks onboarding or incident response
   - P1: Missing and significantly hampers productivity
   - P2: Would improve developer experience
   - P3: Nice to have, low impact
4. **Current State**: What exists today (inline comments? partial README section? nothing? outdated external doc?)
5. **External Coverage**: Link to Glean doc if partial/outdated coverage exists externally
6. **Suggested Outline**: 3-5 bullet points of what the doc should cover
7. **Mermaid Diagram Plan**: Which diagram type(s) and what they should visualize

#### Step 3.3: Mermaid Diagram Type Selection Guide

Follow this mapping when suggesting diagrams:

| Content Type | Recommended Mermaid Diagram | When to Use |
|-------------|---------------------------|-------------|
| System architecture | `graph TD` (flowchart) | Showing components and their relationships |
| Request/data flow | `sequenceDiagram` | Showing how a request flows through services |
| Data pipeline | `graph LR` (left-to-right flowchart) | ETL/ELT steps, data transformations |
| Entity relationships | `erDiagram` | Database schema, data model relationships |
| State machines | `stateDiagram-v2` | Workflow states, status transitions |
| Deployment pipeline | `graph TD` | CI/CD stages, deployment flow |
| Class/module structure | `classDiagram` | Module dependencies, inheritance |
| Decision trees | `graph TD` | Branching logic, configuration decisions |
| Timeline/process | `gantt` | Onboarding steps, migration plans |
| Git workflow | `gitgraph` | Branching strategy |

#### Step 3.4: Generate the GitHub Issue

**IMPORTANT:** Before creating the issue, present a summary to the user and wait for confirmation:

```
I've completed the documentation gap analysis for {repo-name}. Here's what I found:

- {N} documentation gaps across {M} categories
- {N} existing external docs found on Confluence/Glean
- Top 3 critical gaps:
  1. {gap}
  2. {gap}
  3. {gap}

Shall I create the GitHub issue with the full analysis?
```

Once the user confirms, use `gh issue create` with the following structure:

**Issue Title:** `[Docs] Documentation Gap Analysis: {repo-name}`

**Issue Body:**

```markdown
# Documentation Gap Analysis: {repo-name}

> Generated by the `docs-gap-analyzer` skill on {date}.
> Cross-referenced against REA internal knowledge base (Glean).

## Executive Summary

- **Total gaps identified**: {N}
- **Critical (P0)**: {N} | **High (P1)**: {N} | **Medium (P2)**: {N} | **Low (P3)**: {N}
- **Existing external docs found**: {N} (linked below)
- **Estimated documentation effort**: {S/M/L}

## Existing Documentation Inventory

| Location | Document | Status | Notes |
|----------|----------|--------|-------|
| `./README.md` | Project README | {Complete/Partial/Outdated} | {notes} |
| Confluence | {doc title} | {status} | [Link]({url}) |
| ... | ... | ... | ... |

## Documentation Gaps by Category

### {icon} {Category Name}

#### Gap 1: {Topic Title}
- **Priority**: {P0/P1/P2/P3}
- **Current State**: {description}
- **External Coverage**: {None / Partial - [link]}
- **Suggested Outline**:
  - {bullet 1}
  - {bullet 2}
  - {bullet 3}
- **Mermaid Diagrams**:
  - `{diagram type}`: {what it should show}
  - `{diagram type}`: {what it should show}

{repeat for all gaps in this category}

{repeat for all applicable categories}

## Suggested Table of Contents

If all gaps were addressed, the `docs/` folder structure would look like:

    docs/
    ├── README.md                          # Documentation index
    ├── architecture/
    │   ├── overview.md                    # System architecture overview
    │   ├── data-flow.md                   # Data flow and pipeline architecture
    │   └── decisions/                     # ADRs
    │       └── 001-{decision}.md
    ├── api/
    │   └── {api-type}.md                  # API reference
    ├── data-models/
    │   └── schema.md                      # Data model documentation
    ├── guides/
    │   ├── getting-started.md             # Onboarding guide
    │   ├── local-development.md           # Local setup
    │   └── contributing.md                # Contribution guide
    ├── operations/
    │   ├── deployment.md                  # Deployment guide
    │   ├── runbook.md                     # Operational runbook
    │   └── monitoring.md                  # Monitoring and alerting
    ├── configuration/
    │   └── environment.md                 # Configuration reference
    └── testing/
        └── strategy.md                    # Testing strategy

## Recommended Implementation Order

1. {Topic} (P0)
2. {Topic} (P0)
3. {Topic} (P1)
...

## Labels

This issue was auto-generated. Suggested labels: `documentation`, `tech-debt`.
```

Use the GitHub CLI:
```bash
gh issue create \
  --title "[Docs] Documentation Gap Analysis: {repo-name}" \
  --body "{issue body}" \
  --label "documentation"
```

If the `documentation` label does not exist, omit the `--label` flag.

### Phase 4: Self-Review and Quality Check

#### Step 4.1: Completeness Check

Re-scan the codebase for any structural elements that were missed. Specifically verify:
- Did you check ALL config files?
- Did you look at CI/CD pipeline definitions?
- Did you check for infrastructure-as-code?
- Did you look at test fixtures and test utilities?
- Did you check for any `scripts/` or `tools/` directories?

#### Step 4.2: Glean Verification

Perform 2-3 additional Glean searches using alternative keywords to catch external docs that may have been missed in Phase 2:
- Search for the team name + "documentation"
- Search for specific technology terms found in the codebase
- Search for upstream/downstream service names

#### Step 4.3: Priority Validation

Review the priority assignments:
- Is there at least one P0 if the repo has no onboarding guide?
- Are architecture docs P0 or P1 for complex systems?
- Are operational runbooks P0 for production services?

If gaps or corrections are found during self-review, update the issue body before creating it.
