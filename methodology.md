# Methodology

The goal is not to produce a large number. The goal is to produce a number that survives a second reader.

## Unit of analysis

One **job** in one **workflow file** on one **git ref**.

A repository is not the unit. A marketplace action is not the unit. A Code Search hit is not the unit.

## Inclusion test (conjunction)

Mark **in-scope** only if every clause is true.

### 1. Reachability

The job can start from an event an actor without `write` can influence:

- `issues` / `issue_comment`
- `pull_request_target`
- `workflow_run` chained from an untrusted workflow
- public fork `pull_request` **only if** the job still receives secrets or `id-token: write` (default `pull_request` from forks usually does not)

If the only trigger is `push` to a protected default branch by maintainers, stop.

### 2. Gate bypass

At least one of:

- `allowed_non_write_users` is `*` or contains the untrusted actor
- `allowed_bots: '*'` on a public repo (anyone can install an app and appear as `something[bot]`)
- a custom condition that treats comment body / label / title as authorization

Default write-permission check **without** those opt-ins → out of scope for untrusted actors.

### 3. Identity capability

The same job can do at least one of:

- `permissions.id-token: write`
- read a secret (`ANTHROPIC_*`, `CLAUDE_CODE_OAUTH_TOKEN`, cloud keys)
- call `configure-aws-credentials` / `google-github-actions/auth` / equivalent
- use a `GITHUB_TOKEN` with write scopes beyond the minimum needed to post a label

`id-token: write` alone is not theft. It is the **capability to mint**. It becomes interesting only with clauses 1, 2, and 4.

### 4. Exit from the model

The agent is allowed a tool that can:

- read process environment
- run a shell
- open a socket

A review-only agent with no Bash and no env access fails this clause even if the YAML looks noisy.

## Exclusion list (deliberate)

Do not count:

| Pattern | Why it is discarded |
|---|---|
| Fork of a template that still contains the YAML but Actions disabled | No runtime |
| Archived repo | No current operator |
| Action pin to a version that already scrubs OIDC from the agent subprocess, **and** no other secret in-job | Residual depends on remaining tools; do not inflate |
| Same file copied across 800 forks of one chat UI | One template, many mirrors. Report the template once, list forks as copies |
| Name-brand org whose workflow fails any clause | Prestige is not a finding |

## Evidence standard

For each in-scope job record:

```
repo:
workflow:
ref / pin:
trigger:
bypass:
identity:
tools:
last_seen:
notes:
```

No screenshot of Code Search as a substitute for that row.

A public issue comment that contains a unique hostname is not runner telemetry. Notification scanners resolve links. GitHub-hosted runners do not source from consumer DNS resolvers. Discard that class of “proof.”

## What the method is for

- An operator can run it on their org in an afternoon.
- A reviewer can reject a row that fails one clause.
- A portfolio can show judgment: what was kept, what was thrown away.
