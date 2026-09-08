[trail.md](https://github.com/user-attachments/files/31944042/trail.md)
# Research trail

A chronology of *how the question changed*, not a scoreboard.

## 1. Initial question

Can an autonomous agent in GitHub Actions, triggered from a public comment, obtain the job’s OIDC request credentials and mint a token for a cloud audience?

That question is legitimate. It is also older than this repo: it is the subject of [`claude-code-action#1010`](https://github.com/anthropics/claude-code-action/issues/1010).

## 2. First framing error

The first write-up treated three different layers as one bug:

- default write-permission gate inside the action
- documented opt-in that disables that gate
- ambient OIDC credentials on any job with `id-token: write`

A vendor program answers layer 1. An operator lives in layers 2 and 3. Collapsing them produces a report that looks larger than the evidence and closes as Informative.

That close is part of the trail. It is recorded, not relitigated.

## 3. What was discarded

| Claim examined | Outcome |
|---|---|
| Official repo executed attacker tools despite the write gate | Not kept. The run log cited in review failed the gate before tool use. |
| Unique DNS name in a public issue = runner RCE | Not kept. Consumer resolvers and mail scanners resolve public links. |
| Hardening PRs that landed the same week = emergency patch for this report | Not kept. Public timestamps put those PRs before the submission. |
| Code Search hit count = impact | Not kept. Forks of one template are one template. |

Discarding work is the skill. A reader should see that the author can kill their own headline.

## 4. What was kept

- Coupling as the unit: untrusted language + tools + identity in one job.
- Conjunction test with four clauses (see `methodology.md`).
- Double-job split as the control that does not depend on model obedience.
- Credit to prior public work instead of priority theater.

## 5. Current question

Which production jobs, on the ref operators actually pin, still satisfy all four clauses?

That is an audit question. It has a table as an answer. It does not need a severity emoji.
