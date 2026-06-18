# GitHub / GHES Mermaid Rendering Rules

Authoritative reference for what actually renders on GitHub and GHES. Distilled from research conducted 2026-03-24.

---

## Safe Diagram Types

These types render correctly on all supported GHES versions and GitHub.com:

| Type | Keyword |
|------|---------|
| Flowchart | `flowchart LR` / `flowchart TD` |
| Sequence | `sequenceDiagram` |
| Class | `classDiagram` |
| State | `stateDiagram-v2` |
| Entity-relationship | `erDiagram` |
| Gantt | `gantt` |
| Pie | `pie` |
| Git graph | `gitGraph` |

**Use only these types in PR descriptions and automated comments.** Newer types (architecture, kanban, radar, packet, C4) require a recent GHES version — check with `info` before using.

To verify the active Mermaid version on any GitHub instance, post:
````
```mermaid
info
```
````

---

## Character Escaping

Unquoted labels break silently. Wrap any label containing special characters in double quotes.

| Character | Problem | Fix |
|-----------|---------|-----|
| `( )` | Breaks node shape parsing | Wrap label in quotes, or use `#40;` / `#41;` |
| `[ ]` | Conflicts with bracket shape | Wrap in quotes |
| `{ }` | Conflicts with diamond shape | Wrap in quotes |
| `%` | Starts a comment (`%%`) | Wrap in quotes |
| `#` | Entity code prefix | Use `#35;` inside labels, or wrap in quotes |
| `"` | Terminates quoted string | Use `#quot;` inside quoted label |
| `&` | HTML entity | Use `&amp;` or `#38;` |
| `<` `>` | HTML tags | Use `&lt;` / `&gt;` |
| Smart quotes `"` `"` | Break parsing entirely | Replace with straight `"` |
| Em dash `—` / en dash `–` | Not valid syntax | Replace with `-` |
| Unicode arrows `→` `⇒` | Not valid | Use `-->` / `==>` |
| Emojis | Cause rendering errors | Avoid or use HTML entity codes |

**Node IDs must be `[A-Za-z0-9_]+` only** — no hyphens. Hyphens are valid edge syntax and will confuse the parser.

**Safe quoting pattern:**
```
A["label with (parens) and 50% text"] --> B
```

Line breaks in labels: use `<br/>` inside a quoted label.

---

## Size Limits

GitHub uses Mermaid defaults. Users cannot override these via `%%{init}%%`.

| Limit | Value | Effect when exceeded |
|-------|-------|---------------------|
| `maxTextSize` | 50,000 characters | Inline error: "Maximum text size exceeded" |
| `maxEdges` | 500 edges | Inline error: "Too many edges" |

Keep PR comment diagrams under ~50 nodes and ~200 edges for readability.

---

## `<details>` Block Bug

Mermaid diagrams inside a **collapsed** `<details>` block fail when expanded if they have labelled edges. Error: `Cannot read properties of undefined (reading 'x')`.

- Diagrams **without** labelled edges: render correctly after expand
- Diagrams **with** labelled edges: fail — blank or error

**Workaround:** Use `<details open>` (starts expanded) for any diagram with edge labels. Guardian's PR description template already uses `<details open>` for this reason.

---

## Theming

GitHub automatically syncs Mermaid to the viewer's appearance setting:
- Light mode → `default` theme
- Dark mode → `dark` theme

**Do not specify `%%{init}%%` themes in PR comments.** Hardcoding a theme disables this sync — light-theme diagrams will have poor contrast for dark-mode viewers (roughly half of all viewers).

`classDef` node styling and `linkStyle` edge styling are safe to use. Font families are ignored by GitHub's container CSS.

---

## Version History

| Period | GitHub.com version |
|--------|--------------------|
| Jan 2023 | 9.1.6 |
| Aug 2023 | 10.0.2 |
| Feb 2024 | 10.8.0 |
| Apr 2025 | 11.4.1 |

GHES bundles the Mermaid version that shipped with each GHES release — it cannot be upgraded independently. Newer diagram types available on GitHub.com may not be available on your GHES instance.

---

## Flowchart Non-Obvious Patterns

Useful features from [Mermaid flowchart syntax](https://mermaid.js.org/syntax/flowchart.html) that help with layout and readability:

**Invisible links** — `A ~~~ B` creates a spacer between nodes without drawing an arrow. Use this to push nodes apart when they're colliding:
```
A[Step 1] ~~~ B[Step 2]
A --> C[Step 3]
```

**Minimum-length dashes** — extra dashes in an edge force the edge to span more layout ranks, pushing connected nodes further apart:
```
A --> B          %% normal rank distance
A ----> B        %% spans 2 extra ranks
```

**Markdown in node labels** — use `"` with backtick-delimited markdown for bold/italic and auto-wrapping long text (Mermaid 11+):
```
A["`**Bold title**
second line`"]
```
Note: backtick markdown labels require double-quoting the whole node expression. Test on Mermaid Live before using.

**`end` node name** — the word `end` closes a `subgraph` block. If you need a node named "end", capitalize it: `END[End]`.

**Subgraph direction gotcha** — `direction LR` inside a subgraph is **ignored** if any node inside the subgraph has an edge to a node outside it. The subgraph inherits the parent graph direction instead.

**`o---` and `x---` edge variants** — `A o--o B` draws circle arrowheads (bidirectional), `A x--x B` draws cross arrowheads. Useful for showing non-directional or terminated relationships. Note: node IDs starting with `o` or `x` can trigger this syntax accidentally — use CamelCase IDs to avoid.

---

## Platform Notes

- **GitHub mobile** (iOS/Android): does not render Mermaid — shows raw code
- **GitHub Pages** (Jekyll): does not render Mermaid natively — requires a plugin
- **Multiple diagrams in one comment**: all render, but a parse error in any one can cause adjacent diagrams to fail
- **`%%{init}%%` layout hints** (`curve`, `flowchart` config): work where supported, ignored otherwise
