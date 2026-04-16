# Clawndom Agent Template

A minimal skeleton for a Clawndom agent. Fork this, rename, and customize.

## What Clawndom expects

At runtime, Clawndom clones your repo and reads `clawndom.yaml`. Your
agent's logical name comes from Clawndom's `AGENTS_CONFIG` — *not* from
anything in this repo.

```yaml
# Clawndom side (not here — in the deployment's AGENTS_CONFIG):
agents:
  - name: my-agent
    repo: git@github.com:my-org/my-agent.git
    # path:  optional — subdirectory when the repo is a monorepo
    # ref:   optional — branch, tag, or commit SHA to pin
```

## Layout

```
clawndom.yaml            Routing rules and optional model rules
templates/               Nunjucks templates rendered with the webhook payload
docs/                    Policy/context docs pulled in via {{doc:...}}
```

You can add any other directories you want (`identity/`, `specs/`,
`memory/`, etc.) — Clawndom doesn't require them, but they're
conventional places for agent context. Directory structure inside the
repo is entirely up to you; templates reference files by path relative
to this directory.

## Routing

Each rule has a `condition` that evaluates against the webhook payload
and a `messageTemplate` path (relative to this repo) that gets rendered
and sent to the runner.

**Leaf matchers:**

| Form | Example |
| --- | --- |
| `equals` | `equals: { field: issue.fields.status.name, value: Plan }` |
| `in` | `in: { field: issue.fields.status.name, values: [Plan, Planning] }` |
| `matches` | `matches: { field: webhookEvent, pattern: ^comment_, flags: i }` |
| `exists` | `exists: { field: issue.fields.assignee }` |

**Composites:**

| Form | Meaning |
| --- | --- |
| `all_of: [...]` | AND (empty list → true) |
| `any_of: [...]` | OR (empty list → false) |
| `not: <condition>` | negation of exactly one child |

Composites nest freely. See `clawndom.yaml` for a compound example.

## Templates and doc injection

Templates are [Nunjucks](https://mozilla.github.io/nunjucks/). The webhook
payload is the render context — `{{ issue.key }}`, `{{ event.channel }}`, etc.

Inline a file with `{{doc:path/to/file.md}}`. Paths are resolved relative
to the agent directory, so `{{doc:docs/foo.md}}` reads `docs/foo.md` in
this repo. Keep policies in `docs/` and reference them from multiple
templates instead of copy-pasting.

## Model selection

Optional `modelRules` let you pick a stronger model for specific payloads.
Omit the block to use the runner's default.

```yaml
modelRules:
  jira:
    - field: issue.fields.priority.name
      matches: ['Highest', 'High']
      model: claude-opus-4-6
```

## First match wins — across rules and across agents

Clawndom walks `AGENTS_CONFIG` in order, then each agent's rules in
order. The first rule whose `condition` evaluates to true wins — the
event routes to that agent with that template. If nothing matches
anywhere, Clawndom logs `routing:no-match` and skips forwarding.

Use a catch-all rule (`condition: { all_of: [] }`) at the bottom of an
agent's list if you want that agent to be the fallback for a provider.
