# approval-builtin-copilot

A small todo app (Spring/Maven backend, React/Vite client) used to show one way to approve pull requests: **GitHub's built-in Copilot approvals**, configured with settings only and no custom code.

> [!IMPORTANT]
> **Code-owner review is turned off in this repository.**
> Copilot's approval cannot satisfy a code-owner requirement, so with it turned on every pull request would still need a human.
> As a result, `CODEOWNERS` in this repository only requests reviewers. It does **not** block merging.

## Results

_Pending: filled in after the three pull requests have been reviewed._

## Running locally

Backend (JDK 21, Maven):

```bash
cd backend && mvn verify
```

Client (Node 22):

```bash
cd client && npm ci && npm run dev
```
