# RoleSync — Workflows

High-level description of the three role journeys the platform implements. This document is
conceptual: it describes *what the system does and which rules it enforces*, not how the code
is written.

Every rule below is enforced on the server. The web client mirrors some of them for
usability, but no client-side decision is ever authoritative.

---

## Candidate

```
Register  →  Verify email (OTP)  →  Sign in  →  Build profile  →  Upload resume
                                                       │
                                                       ▼
              Search jobs  →  Filter  →  View job  →  Apply  →  Track application status
```

**Register.** Email and password. The password is hashed before storage; a password policy is
applied on the server, and the client mirrors it only to save a round trip.

**Verify email.** A one-time code is emailed. It is stored hashed, expires, is consumed on
first successful use, and is protected by an attempt cap and a resend cooldown. Verifying
contact details proves control of an email address — nothing more.

**Profile and resume.** Skills, education and experience. Resumes are validated on type and
size, stored privately under a generated filename, and are readable only by the owning
candidate, by a recruiter whose own job the candidate applied to, or by an administrator for
moderation.

**Search.** Keyword, location, engagement type and salary range, paginated. Only active jobs
from companies that completed verification are ever visible.

**Apply.** One application per candidate per job, enforced by both a database constraint and
a service-level check. The application permanently keeps the resume that was attached at the
moment of applying — uploading a newer resume later never rewrites an application a recruiter
has already reviewed.

**Track.** The candidate sees the current status and its history. A candidate can create an
application but can never advance it.

---

## Recruiter / Company

```
Register  →  Verify email (OTP)  →  Create company profile  →  Upload verification documents
                                                                        │
                                                    (administrator review)
                                                                        ▼
                     Post jobs  →  Manage jobs  →  Review applicants  →  Move applications
```

**Company profile and verification.** The recruiter submits company documents for review.
Each submission or resubmission is recorded as a new entry rather than overwriting the last
one, so the review history survives across rounds.

**The publish gate.** A recruiter may publish an active job only while *all three* of these
hold at the same time:

- the account is active,
- contact verification has passed,
- company verification has been approved.

All three are read live from the database at the moment of the request. They are never
collapsed into one stored flag and never taken from a token claim. A recruiter who fails any
one of them cannot publish, and the API says which condition is unmet.

**Verification cascades.** Two events reverse a company's approved status, and each is a
single all-or-nothing transaction that also deactivates that company's live jobs:

1. the recruiter edits a verification-sensitive company field while approved, which sends the
   company back for re-review;
2. an administrator suspends the company's verification.

Restoring a suspended company is a *separate, simpler* transaction that never touches job
rows. Bringing a job back is always a later, explicit recruiter action, re-checked against
the full publish gate above. Restoration also returns the company to whatever status actually
applied before the suspension, rather than assuming it was approved.

**Applicants.** A recruiter sees applicants only for their own jobs. Ownership is resolved
from the authenticated identity, never from an identifier supplied in the request.

---

## Administrator

```
Sign in  →  Dashboard  →  Review company verifications  →  Approve / Reject / Request resubmission
                    │
                    ├──  Manage users (suspend / reactivate)
                    ├──  Moderate jobs and applications
                    └──  Read the audit log
```

There is no public registration route for this role. The administrator account is provisioned
from configuration.

**Verification review.** Approve, reject, or request a resubmission. Document review has its
own status vocabulary, deliberately kept distinct from the company-level verification status
so the two cannot be confused.

**Audit log.** Append-only. One entry per consequential action: verification decisions, user
suspension and reactivation, job deactivation, the automatic re-review triggered by a
sensitive company edit, and administrator suspend/restore — including one entry per job that
a cascade deactivated.

---

## Application status machine

The hiring pipeline is a server-controlled state machine. Only the recruiter who owns the job
may move an application through it, and every transition appends an immutable history entry
recording who changed what, when.

```
APPLIED     ─→  SHORTLISTED  ─→  INTERVIEW  ─→  OFFERED  ─→  HIRED   (terminal)
    │               │               │              │
    └───────────────┴───────────────┴──────────────┴──────→  REJECTED (terminal)
```

Any transition not drawn above is rejected by the server. `HIRED` and `REJECTED` are terminal
— nothing moves out of them.

---

## Cross-cutting rules

**Deletion.** Where history matters — jobs, users, applications — the platform changes status
instead of deleting rows, so applications and audit history stay readable after a job or
account is taken out of circulation.

**Separation of verification concerns.** Account status, contact verification and company
verification are three independent fields. Passing an email OTP is never treated as evidence
that a business is legitimate.

**File access.** Verification documents are visible to the owning recruiter and authorized
administrators only — never to candidates or the public. Storage paths supplied by a client
are never treated as authorization.
