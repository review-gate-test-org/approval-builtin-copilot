# approval-builtin-copilot

A small todo app (Spring/Maven backend, React/Vite client) used to show one way to approve pull requests: **GitHub's built-in Copilot approvals**, configured with settings only and no custom code.

> [!IMPORTANT]
> **Code-owner review is turned off in this repository.**
> Copilot's approval cannot satisfy a code-owner requirement, so with it turned on every pull request would still need a human.
> As a result, `CODEOWNERS` in this repository only requests reviewers. It does **not** block merging.

## What this path shows

Copilot reviews every pull request automatically. When it approves, its approval counts as the one required approval, but only if **every** changed file matches the allowed paths (`backend/**`). Any pull request that touches another path needs a human approval.

Status labels used below: **verified** means observed in this repository on 2026-10-29. **Unverified** means taken from GitHub docs and not tested here.

## Results

| PR | Change | Copilot verdict | Merge state | What the PR page shows |
| --- | --- | --- | --- | --- |
| A: [#1](https://github.com/review-gate-test-org/approval-builtin-copilot/pull/1) | backend only, benign | 🟢 Approved | Mergeable, with no human review | Merge button enabled |
| B: [#2](https://github.com/review-gate-test-org/approval-builtin-copilot/pull/2) | backend only, off-by-one bug | 🟡 Changes recommended (1 high-severity comment) | Blocked | Review required |
| C: [#3](https://github.com/review-gate-test-org/approval-builtin-copilot/pull/3) | backend + `README.md` | 🟢 Approved | Blocked | "Review required. At least 1 approving review is required by reviewers with write access." The page shows Copilot's approval, but it does not count. |

All three rows are **verified**: the API reports `reviewDecision` and `mergeStateStatus`, and the PR page was checked by hand. The ruleset has no bypass actors, and the API reports `viewerCanMergeAsAdmin=false` for these pull requests.

The pull requests stay open and are never merged.

Two further observations, both **verified**:

- **An empty path list allows every path.** In the first run the "Limit approvals to certain file paths" box was accidentally left empty. Copilot's approval then made C mergeable without a human. If the scope box is left empty or cleared, the human gate silently disappears for every path.
- **The path scope is checked when merging, not when Copilot reviews.** After `backend/**` was saved, C changed from mergeable to blocked without a new review.

## How it is configured

- **Ruleset** [`main: Copilot or human approval`](https://github.com/review-gate-test-org/approval-builtin-copilot/rules/26113860), active on the default branch, with no bypass actors:
  - pull request required with 1 approval
  - stale approvals dismissed on push
  - **code-owner review not required**
  - required checks `backend` and `client` (GitHub Actions, [`ci.yml`](.github/workflows/ci.yml))
  - Copilot code review requested automatically on every push
  - deletion and force-push blocked
- **Repository settings** (Copilot → Code review → Approvals), which can only be changed in the UI. GitHub does not document an API for them:
  - Allow Copilot to approve pull requests: **on**
  - Copilot approvals can count toward merge requirements: **on**
  - Limit approvals to certain file paths: `backend/**`
  - Review effort: default (Balanced)
- **Organization setting**: "Count Copilot approvals toward merge requirements" is set to *Let repositories decide*.
- **[`CODEOWNERS`](.github/CODEOWNERS)**: `/backend/` belongs to `@review-gate-test-org/backend`, `/client/` to `@review-gate-test-org/frontend`, and everything else to `@venetrius`. Because code-owner review is off, these entries only request reviewers.

## Limits

- **No code-owner approval (verified).** Copilot's approval cannot satisfy a code-owner requirement. To let Copilot approve alone, code-owner review has to be turned off for the whole branch. Any approval from Copilot or from a human with write access then counts, whoever owns the code.
- **Coarse path scope (verified for one glob).** It is a single list of up to 15 globs for the whole repository (the 15 limit is from the docs, unverified here). It applies to the whole pull request: one file outside the list means Copilot's approval does not count. There is no per-team or per-path policy, and no way to say "Copilot may approve backend, but the frontend team must approve client."
- **Only Copilot (by design, per GitHub docs).** The only automated approver this path supports is GitHub's own Copilot code review. You cannot plug in your own review agent, policy, or audit trail.
- **The verdict is non-deterministic (unverified).** The same change may get a different verdict on another run. Each scenario was tested once.
- **New pushes dismiss Copilot's approval (per GitHub docs, unverified here).** Copilot then has to review again.

## Sources

- [Changelog: Copilot code review can now approve pull requests](https://github.blog/changelog/2026-09-01-copilot-code-review-can-now-approve-pull-requests/) (public preview, 2026-09-01)
- [Changelog: Copilot approvals are generally available](https://github.blog/changelog/2026-10-28-copilot-code-review-merge-with-confidence-with-max-and-approvals/) (2026-10-28)
- [Docs: Configuring code review by GitHub Copilot](https://docs.github.com/copilot/how-tos/copilot-on-github/set-up-copilot/configure-code-review)

## Running locally

Backend (JDK 21, Maven):

```bash
cd backend && mvn verify
```

Client (Node 22):

```bash
cd client && npm ci && npm run dev
```
