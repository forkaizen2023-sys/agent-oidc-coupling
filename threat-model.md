# Threat model

## Assets

| Asset | Why it matters |
|---|---|
| GitHub OIDC request pair (`ACTIONS_ID_TOKEN_REQUEST_TOKEN` + `_URL`) | Lets the job mint a JWT for a configured audience. |
| Cloud role trust (`sts:AssumeRoleWithWebIdentity`, GCP WIF, Azure federated creds) | The JWT becomes cloud credentials. |
| `GITHUB_TOKEN` / GitHub App installation token | Write to the same repo: code, workflows, releases. |
| Long-lived agent credentials (`CLAUDE_CODE_OAUTH_TOKEN`, API keys) | Survive the job if exfiltrated. |
| Build integrity | A write token plus RCE-class tools can change what the next pipeline runs. |

## Actors

| Actor | Assumption |
|---|---|
| Maintainer with write | In scope for vendor default model. Out of scope here. |
| External GitHub user | Can open issues and comments on a public repo. |
| External GitHub App | Can appear as `name[bot]` if installed on a repo the actor controls. |
| Contributor without write | Can open PRs; dangerous mainly on `pull_request_target`. |

This model does **not** assume a bypass of a correctly enforced write gate. It assumes operators copy a documented opt-in into a job that still holds identity.

## Attack path (architectural)

1. Actor places instructions in an issue, review, or title.
2. Workflow starts because the trigger is public and the gate is relaxed.
3. Agent runs with tools in the same environment that can mint or read credentials.
4. Agent is steered into using those tools.
5. Identity leaves the job, or the job writes back into the repo.

Step 4 is probabilistic (model behavior). Steps 1–3 and 5 are deterministic if the YAML says so. Design controls target the deterministic part.

## Controls, ranked

1. **Do not put identity in Job A.** Highest leverage. See `examples/double-job-review.yml`.
2. **Do not relax the write gate** on any job that has identity.
3. **Pin the action** and keep OIDC request variables out of the agent subprocess (upstream hardening exists; do not treat it as optional).
4. **Shrink tools.** No Bash if the task is labeling.
5. **Fail closed** if sandbox dependencies cannot load.
6. Prompt text that says “ignore injections.” Lowest leverage. Keep it; do not rely on it.

## Residual risk after double-job

- Job A can still produce a misleading review. Humans must not treat the artifact as authority.
- Job B can still be tricked if it blindly executes the artifact as code. Job B must treat the artifact as data.
- A later workflow that consumes the artifact with secrets re-creates the coupling. Track data flow across jobs, not just inside one file.

## Vendor vs operator

Vendor programs model the action as shipped.  
Operators model the workflow as copied.

Both can be correct at once. This document is written for the second reader.
