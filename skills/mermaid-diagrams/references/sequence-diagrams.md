# Sequence Diagrams — GitHub/GHES Edition

`sequenceDiagram` is on the GHES-safe list. Use it in standalone platform documentation to show step-by-step interactions between system components — not in PR description diagrams (those should use `flowchart LR`).

---

## Core Syntax

```mermaid
sequenceDiagram
  actor User
  participant Frontend
  participant API
  participant DB

  User->>Frontend: click submit
  Frontend->>+API: POST /resource
  API->>+DB: query
  DB-->>-API: result
  API-->>-Frontend: 200 OK
  Frontend-->>User: success message
```

**Participants:**
- `participant Name` — system component (service, workflow, API)
- `actor Name` — external entity (user, external system)
- `participant Short as "Long Label"` — alias for long names

**Arrow types:**
- `A->>B: message` — solid, synchronous call
- `A-->>B: message` — dashed, return/response
- `A-)B: message` — solid, async (fire-and-forget)
- `A-xB: message` — solid with X (delete/terminate)

**Activations** (show when a participant is actively processing):
- `A->>+B:` activates B
- `B-->>-A:` deactivates B

---

## Conditional Logic

```mermaid
sequenceDiagram
  Client->>API: authenticate
  alt valid token
    API-->>Client: 200 OK
  else expired token
    API-->>Client: 401 Unauthorized
  end
```

- `alt / else / end` — conditional branches
- `opt / end` — optional block (executes 0 or 1 times)
- `loop label / end` — repeated block
- `par / and / end` — parallel execution

---

## Guardian Example: @guardian mention flow

```mermaid
sequenceDiagram
  actor Dev as Developer
  participant GH as GitHub
  participant GW as guardian-agent.yml
  participant CC as claude-code-action

  Dev->>GH: comment "@guardian please review auth changes"
  GH->>GW: issue_comment event
  GW->>GW: build --allowedTools list
  GW->>+CC: invoke with comment as prompt
  CC-->>-GW: review output
  GW->>GH: post comment reply
  GH-->>Dev: notification
```

---

## GitHub-Safe Notes

- **No clickable links** — `link Participant: Label @ URL` syntax is documented in Mermaid but does not work on GitHub; links and tooltips are stripped
- **No `autonumber` issues** — works fine, use it for complex flows where message ordering matters
- **Notes work:** `Note over A,B: text` and `Note right of A: text` both render correctly
- **Keep participants ≤ 6** — more than 6 becomes unreadable at GitHub's fixed PR comment width
- **`create participant` syntax** requires Mermaid 10.3+, available on GitHub.com but check GHES version
