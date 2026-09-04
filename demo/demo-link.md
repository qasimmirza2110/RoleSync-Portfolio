# RoleSync Demo

RoleSync is currently presented through this public portfolio repository.

**A live production deployment is not currently available.** There is no hosted URL, no demo
login, and no publicly reachable API. Nothing in this repository should be read as pointing
at a running environment.

## What is available here instead

| | |
|---|---|
| **Screenshots** | [`../screenshots/`](../screenshots/) — captured from the running application against a local synthetic dataset |
| **Architecture** | [`architecture.png`](../docs/architecture.png) — the full request path from client to database |
| **Workflows** | [`workflows.md`](../docs/workflows.md) — the three role journeys and the rules the server enforces |
| **Feature and engineering detail** | [`../README.md`](../README.md) |

## Why there is no hosted demo

The platform is a single Spring Boot service backed by MySQL, with private file storage for
resumes and company verification documents. A public demo would require a hosted database and
an object store holding uploaded documents, and would invite exactly the kind of unverified
signup the platform is designed to gate. That was not a trade worth making for a portfolio
piece.

## Running it locally

The full source, including setup instructions, lives in a private repository. Access can be
granted on request for interviews or technical review — see **Repository Access** in the
[README](../README.md).

## Verification status

The screenshots in this repository were taken from the application actually running locally
against a synthetic demo dataset. They are not mockups, and they are not design comps.

Authenticated dashboard screens (candidate, recruiter and administrator) are **not** shown
here, because capturing them requires signing in with real account credentials. The README
says so plainly rather than substituting mockups for them.
