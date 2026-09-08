RECORD — sealed 2026-09-08

This file is the stop-rule for the investigation. If a later sentence contradicts this file, the later sentence is wrong.

Author: David Alfonso Jaimes Olivo
Identity used in disclosure: forkaizen / forkaizen2023-sys
Object: agent + GitHub Actions + workload identity
Vendor program touched: Anthropic VDP via HackerOne (April 2026)



FACT





An autonomous agent that shares a GitHub Actions job with id-token: write, secrets, or a cloud role can be steered toward minting or moving identity, if it also has a tool that reads the environment or opens a network path.



anthropics/claude-code-action enforces a write-permission check before tools run. That is the vendor default threat model.



allowed_non_write_users (including *) is a documented opt-in that relaxes that check. Official example workflows use it for issue triage. Operators who copy it into a job that still holds identity create the coupling.



Ambient OIDC request variables on jobs with id-token: write were described publicly in anthropics/claude-code-action#1010 (chyipin, March 2026). Upstream later scrubbed those variables from the agent subprocess. That work predates the April VDP submission and is not this author's discovery.



A VDP report on this class was submitted 8 April 2026 and closed Informative. After dispute, the program maintained the close: the cited official run failed the write gate before tool use; DNS lookups from a public issue are not runner execution; cited hardening PRs predated the report; downstream opt-in copies are out of vendor program scope.



Code Search hit counts, forks of one template, and brand names without a four-clause row are not impact.

INFERENCE

The investigation established an operator threat model and a privilege-split control (Job A analyzes without identity; Job B publishes or assumes without a model).

It did not establish a validated bypass of the action's default write gate. It did not establish vendor acceptance of a Critical. It did not establish that subsequent version bumps were a response to this report.

Vendor scope and operator scope are different questions. Both answers can stand.

OUT OF SCOPE — do not reopen





Relitigating the Informative close as a status trophy.



Treating mail-scanner / public-resolver DNS as proof of RCE on GitHub-hosted runners.



Publishing “N repos vulnerable” from a search screenshot.



Naming third-party orgs as victims without a current four-clause row on the pinned ref.



Invented CVE IDs, signature hashes, or formulas as substitutes for the record above.

METHOD KEPT

A job is in-scope only if all four hold on the same job, on the ref operators pin:





Trigger reachable without write



Gate bypass (* or equivalent)



Identity capability in that job



Agent tool that can leave the model

Fail one clause → discard.

CONTROL KEPT

Double-job pattern. Example: examples/double-job-review.yml.

STOP-RULE

This cycle is closed. New work starts only with a table of four-clause rows, or with a different target. Doubt about “what I investigated” is answered by this file, not by the HackerOne thread.

Sealed: 2026-09-08
