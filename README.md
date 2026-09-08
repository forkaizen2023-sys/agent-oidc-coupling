# Agent–OIDC Coupling in GitHub Actions

**Operator threat model** for autonomous coding agents that share a job with workload identity.

This repository documents a research trail: how the problem was framed, what was measured, what was discarded, and a control that does not depend on a vendor severity label.

It is not a vulnerability advisory for `anthropics/claude-code-action`.  
It is not a bounty write-up.  
It is a method someone else can rerun.

---

## Thesis

Prompt injection is not the interesting failure.

The interesting failure is **privilege continuity**:

```
untrusted text  →  agent with tools  →  same process tree as
(issue / PR / comment)                 GITHUB_TOKEN | OIDC | cloud role
```

If those three sit in one job, a comment is no longer text. It is a path to minting identity.

Remove any edge and the class collapses. That is the whole model.

## What this is not

- Not a claim that Anthropic accepted a Critical or patched in response to this work.
- Not a count of “N repositories vulnerable.” Search hits are not takeovers.
- Not evidence that a DNS canary on a public issue is code execution on a runner.
- Not legal advice and not an exploit kit.

A VDP report on this class was submitted to Anthropic in April 2026 and closed **Informative**: the action’s default write-permission gate is the vendor threat model; relaxing it with `allowed_non_write_users` is a documented opt-in; downstream copies of that opt-in are an operator problem. That close is accepted here as the vendor record. The operator problem remains.

## Related work (credit first)

The class was already visible before this note:

| Source | What it established |
|---|---|
| [`claude-code-action#1010`](https://github.com/anthropics/claude-code-action/issues/1010) (chyipin, Mar 2026) | Agent session can inherit `ACTIONS_ID_TOKEN_REQUEST_*` and mint OIDC tokens when `id-token: write` is set. |
| Upstream [security.md](https://github.com/anthropics/claude-code-action/blob/main/docs/security.md) | `allowed_non_write_users: '*'` is documented as a significant risk. |
| Official issue-triage examples | The opt-in exists because triage must run on issues opened by strangers. |
| Trail of Bits [`agentic-actions-auditor`](https://github.com/trailofbits/skills/tree/main/plugins/agentic-actions-auditor) | Static checks for wildcards, dangerous tools, and attacker-controlled input into agent steps. |

This repo adds one thing those sources do not center: **identity-aware conjunction**. Not “does the workflow mention the action,” but “can this job turn a comment into a cloud session.”

## Method

Full procedure: [`docs/methodology.md`](docs/methodology.md).

Short version — a workflow is in-scope only if **all four** hold on the same job, on the ref people actually pin:

1. **Reachable trigger** — `issues`, `issue_comment`, `pull_request_target`, or equivalent path for an actor without write.
2. **Gate bypass** — `allowed_non_write_users` includes `*`, or an equivalent (`allowed_bots: '*'`, auto-trust of `*[bot]`).
3. **Identity capability** — `id-token: write`, a repo/org secret, or a cloud role assumption in that job.
4. **Agent tool that can leave the box** — env read, shell, or outbound network.

Fail one clause → out of scope. That filter is the research product.

## Control

Split authority. The model is a parser. It must not be an issuer.

```
Job A  untrusted analysis     contents: read only
       no id-token, no cloud role, no PAT
       writes an artifact

Job B  trusted publication    needs: A
       posts the comment / opens the PR / assumes the role
       no LLM
```

Reference workflow: [`examples/double-job-review.yml`](examples/double-job-review.yml).

If Job A is compromised, the attacker gets a draft. Not an IAM session.

Threat model write-up: [`docs/threat-model.md`](docs/threat-model.md).

## How to read this repo

| Path | Role |
|---|---|
| `README.md` | Claims and non-claims |
| `docs/methodology.md` | How the surface was filtered |
| `docs/threat-model.md` | Assets, actors, controls |
| `examples/double-job-review.yml` | Control you can copy |
| `docs/trail.md` | Chronology without grievance |

## Use

- Audit your own org’s agent workflows against the four-point test.
- Pin actions. Do not copy `allowed_non_write_users: '*'` into a job that can mint identity.
- Prefer Job A / Job B over prompt-only hardening.

If you want that audit done on a private surface, contact is on [the research site](https://forkaizen2023-sys.github.io).

## License

Documentation: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).  
Example workflows: MIT.
