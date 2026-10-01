# Specification Review: Clarifications Required

**Reviewed**: `spec/constitution.md` and `spec/specification.md`
**Review date**: 2026-10-02
**Readiness**: Suitable as a product draft; not yet precise enough for an
implementation contract or pilot activation.

## Direct Normative Mismatch

### C-001 - Changed-line evidence strength

**Where**: Constitution, “Evidence Before Findings”; specification FR-014 and
FR-015.

The constitution requires every finding to cite a changed line. FR-014 requires
a changed-line range, but FR-015 says inline locations SHOULD map to changed
lines. The different MUST/SHOULD strength leaves room for publishing a finding
that violates the constitution.

**Clarify**: Must every publishable finding map to a changed line, with
otherwise-valid candidates withheld? Define how multi-line findings and cases
where the underlying risk is only in unchanged context are handled, then align
FR-014/FR-015 with the constitutional MUST.

## High-Priority Clarifications

### C-002 - Shadow and comment result surfaces

**Where**: User Story 4, FR-016, FR-017, Dependencies, and Rollout.

The shadow destination is unresolved, yet acceptance scenarios depend on an
“approved” CI/check surface. It is also unclear whether an artifact or dashboard
is required, who can read results, and whether content is exposed to all PR
participants.

**Clarify**: Select the initial surface, its audience and access control, and
what the check displays. Define check conclusions for clean, findings,
partial, skipped, and inconclusive reviews. Confirm that no conclusion becomes
a required branch-protection gate in the initial release.

### C-003 - Finding contract and severity taxonomy

**Where**: FR-014, FR-015, Key Entities, and Proposed Finding Format in the
source specification.

The required fields are listed, but valid category and severity values are not
defined. Confidence may be a score or band; evidence-reference format,
multi-line locations, and the meaning of a stable finding key are unspecified.

**Clarify**: Publish the finding schema and versioning policy. Define category
and severity enums, confidence representation, evidence reference format,
line-range rules, remediation optionality, and the key inputs used for
deduplication across revisions and model versions.

### C-004 - Quality thresholds and comment activation gate

**Where**: FR-017, SC-008 through SC-010, User Story 5, and Rollout.

The specification requires findings “above” a confidence threshold and requires
shadow results to meet an agreed quality threshold, but supplies no values,
measurement window, minimum sample size, or approval owner for the thresholds.

**Clarify**: Define initial thresholds for confidence, precision/actionability,
false-positive rate, and valid-line anchoring, plus minimum evaluation volume
and review period. State who approves each threshold and whether it is global,
organization-level, category-specific, or repository-specific.

### C-005 - Authoritative domain rules

**Where**: FR-007 and FR-009, Assumptions, and Risks and Open Decisions.

Commerce checks are explicitly candidates, not universal invariants. No
authoritative rules, exceptions, service ownership boundaries, event guarantees,
or source-of-truth references are supplied, so the first implementation cannot
determine when a candidate rule applies.

**Clarify**: Identify domain owners and the versioned source for each invariant.
Define how a rule is enabled, scoped, reviewed, and tested, and require the
service to suppress it when applicability cannot be established.

### C-006 - Repository configuration schema and fallback behavior

**Where**: FR-021 through FR-023, Repository Policy, User Story 6, and the
source specification’s Repository Configuration section.

The YAML filename, schema version, required fields, precedence between
organization defaults and repository overrides, and exact safe defaults are
not defined. “Fall back to safe organization defaults” could mean analyze under
defaults or skip analysis; those outcomes have different risk.

**Clarify**: Select the config path and publish a versioned schema covering
categories, path rules, stack hints, domain rules, data classifications,
thresholds, and mode. Define precedence and invalid/missing-config behavior,
including whether review is skipped, which diagnostic is visible, and which
controls repository policy can never override.

### C-007 - Context scope and incomplete reviews

**Where**: FR-005 through FR-008, NFR-003, User Story 2, and Context Source.

“Relevant surrounding files,” “bounded set of affected dependencies,” and
“material partial or skipped coverage” have no selection algorithm or
operational definition. The supported file types, repository layouts, and
handling of unsupported or mixed-language pull requests are also unclear.

**Clarify**: Define supported file types and versions, generated/binary/vendor
file behavior, dependency traversal boundaries, and how references are resolved
at a pinned revision. Set file, diff, token, and traversal limits. Define when
coverage is partial or the review is skipped and how that limitation is
reported.

### C-008 - Webhook, rerun, and stale-review lifecycle

**Where**: FR-003/FR-004, User Story 2, and Review Job.

The trigger list is defined, but the system behavior for draft pull requests,
closed/merged pull requests, reopened jobs, force pushes, manual reruns, and
concurrent revisions is not. It is unclear whether obsolete jobs are cancelled
or merely prevented from publishing.

**Clarify**: Define the supported event/action matrix and a canonical review
identity. Specify cancellation and publication rules for closed PRs and
superseded heads, manual rerun semantics, and how old comments/checks are
updated or withdrawn when findings disappear.

### C-009 - Queue delivery and retry semantics

**Where**: Assumptions, NFR-003, FR-019, and Rollout.

Queue technology is intentionally unselected, but durability, delivery
semantics, deduplication storage, per-repository fairness, concurrency limits,
and webhook acknowledgement behavior are unspecified. “Bounded retries and
backoff” has no retry budget, timeout, or terminal state.

**Clarify**: Set webhook response-time expectations, at-least-once or other
delivery semantics, retryable error classes, backoff/deadline limits, job
timeouts, cancellation, and dead-letter/replay handling. Specify the persistence
boundary between PostgreSQL, the queue, and worker state.

### C-010 - GitHub permissions, operator roles, and rate limits

**Where**: FR-001/FR-002, NFR-001, User Story 1 and User Story 6.

Least privilege is required, but exact App permissions, credential lifecycle,
rate limiting, and authorization roles for configuration, audit data, feedback,
and comment-mode activation are absent. The specification does not define who
can opt a repository in or out.

**Clarify**: Enumerate minimum read/check/comment permissions and separate
shadow/comment permissions where feasible. Define installation authorization,
credential rotation/revocation, API rate-limit behavior, and operator roles for
policy edits, mode changes, audit access, and feedback.

### C-011 - Redaction, model boundary, and output safety

**Where**: FR-010 through FR-012, NFR-002, and Security, Privacy, and
Governance in the source specification.

Redaction is required before model calls, but detector coverage, failure policy,
and treatment of secrets in model output are undefined. A finding or inline
comment could repeat sensitive source content even when the request was
redacted. Prompt-injection handling is stated as a principle but not testable
controls.

**Clarify**: Define secret/PII classes, detectors, custom patterns, false
positive/negative handling, and whether any redaction failure blocks inference.
Require output scanning before storage or publication. Specify model network
allowlisting, tool-call prohibition, prompt-injection test cases, and behavior
when the model endpoint or processing region is not approved.

### C-012 - Data classification, storage, and retention

**Where**: FR-024/FR-025, NFR-001/NFR-002, Audit Event, and Risks and Open
Decisions.

PostgreSQL is mandated, but the persisted data classes and lifecycle are not.
The documents do not say whether diffs, prompts, model traces, findings,
comments, labels, or redaction metadata are stored, for how long, or how deletion
works. “No raw source in logs” does not settle source retention elsewhere.

**Clarify**: Define data inventory, classification, storage location, retention
and deletion periods, backup treatment, legal holds, and access controls for
each data class. State whether source/context is ever persisted and how
organization/repository deletion is honored.

### C-013 - Audit metadata versus data minimization

**Where**: FR-024/FR-025, NFR-004, and Audit Event.

The specification requests actor/source identity and extensive review metadata,
while requiring unnecessary PII to be excluded. It does not identify which
human identity is recorded, whether access/config changes are audited, or who
can inspect audit records.

**Clarify**: Define required actor identifiers, audit events, redaction rules,
retention, access roles, and whether audit records may contain finding text or
repository names. Distinguish immutable security audit data from operational
metrics and feedback.

### C-014 - CI and GitHub publication behavior

**Where**: FR-016 through FR-020, User Stories 4/5, and Initial Pilot
Acceptance Criteria.

The result surfaces are not specified enough to test branch protection or user
experience. “Update or resolve” comments does not state how comments behave
when a finding disappears, moves, changes severity, or is produced by a new
prompt/model version.

**Clarify**: Define check-run names, conclusions, summaries, links, and whether
artifacts are used. Specify inline versus review-level comments, comment
identity/ownership, stale-thread resolution, edits/deletions, rerun behavior,
and permissions. Explicitly state that no result is a required status check
during the initial release.

## Medium-Priority Clarifications

### C-015 - Application user workflows and frontend scope

**Where**: Users and Stakeholders, Technology Constraints, and User Stories.

React 18/Vite is mandated, but no user-facing frontend workflow is specified.
It is unclear whether the frontend is an operator console, evaluation dashboard,
configuration editor, or part of the product at all. Authentication, SSO, and
which roles use it are not described.

**Clarify**: Identify the frontend’s first-release purpose and required screens
or explicitly mark it out of scope. Define authentication, authorization, and
whether setup, feedback, metrics, and mode changes happen in GitHub, the
frontend, or both.

### C-016 - Evaluation protocol and measurable success gates

**Where**: FR-026, SC-008/SC-009, Evaluation and Success Measures, and Rollout.

The required metrics are named but lack formulas, baselines, sample sizes,
segmentation rules, label adjudication, and pass/fail thresholds. SC-009 refers
to an agreed threshold without defining how it is reached. “Coverage” and
“actionability” are not operationalized.

**Clarify**: Define the labeled corpus composition, train/tune/evaluate
separation, adjudication process, formulas for precision and false positives,
denominators, confidence intervals if required, minimum sample size, reporting
cadence, and explicit promotion/rollback gates.

### C-017 - Performance and capacity targets

**Where**: NFR-005 through NFR-007 and Risks and Open Decisions.

The 5-minute p95 and 99.5% availability values are provisional. The population
of “supported pull requests,” measurement period, start/end timestamps,
maintenance exclusions, workload, peak concurrency, and treatment of partial
or skipped reviews are undefined.

**Clarify**: Confirm or replace the targets after measuring pilot volume. Define
the latency clock, availability SLI and denominator, workload profile, peak
concurrency, and whether queue delay, GitHub/model outages, and partial reviews
count against each target.

### C-018 - Failure outcomes and operator response

**Where**: FR-019/FR-020, NFR-003, User Story 4, and Audit/Observability.

The service must be observable and non-blocking, but there are no required
alerts, dashboards, error categories, partial-result states, operator runbooks,
or recovery objectives. It is unclear how users distinguish “no findings” from
“review did not complete.”

**Clarify**: Define terminal review states and user-visible messages; metrics,
logs, alerts, and ownership; retry exhaustion and replay procedures; and
recovery objectives. Ensure a clean review is distinguishable from skipped,
partial, failed, and inconclusive outcomes.

### C-019 - Database and migration expectations

**Where**: Technology Constraints, Modular Contracts and Change Safety, and
Key Entities.

PostgreSQL 15 and reviewed migrations are required, but no ownership or
consistency rules are specified for review jobs, findings, publication state,
audit events, labels, and configuration versions. Backup/restore and schema
compatibility expectations are also absent.

**Clarify**: Define which records are authoritative in PostgreSQL, transaction
and idempotency boundaries, retention/deletion interaction, backup/restore
requirements, and how migrations remain compatible with deployed API/worker
versions.

### C-020 - Dependency and API contracts

**Where**: Constitution, “Modular Contracts and Change Safety”; FR-005,
FR-013, and Dependencies.

The constitution requires explicit contracts for frontend, API, worker, model,
and persistence boundaries, but the feature specification does not identify
API consumers, contracts, compatibility policy, or versioning ownership.

**Clarify**: Identify the first-release API surfaces and their consumers, the
contract/schema formats, compatibility and deprecation rules, and how model
provider and GitHub adapters are tested against those contracts.

### C-021 - Scope and pilot enrollment

**Where**: Scope, FR-008/FR-009, User Stories, and Rollout.

The target is described as an organization of approximately 100 people, but the
number of pilot repositories, teams, PR volume, opt-out owner, and expansion
criteria are absent. It is unclear whether mixed changes containing unsupported
languages receive partial review or are skipped.

**Clarify**: Define pilot cohort and enrollment/opt-out ownership. Set supported
language and infrastructure versions, mixed-language behavior, and the
repository/volume limits for each rollout phase.

### C-022 - Administrative ownership and approvals

**Where**: Users and Stakeholders, User Stories 5/6, SC-009, and Governance.

Security, privacy, domain, and quality owners are referenced but no accountable
roles or approval sequence is specified. It is unclear who can authorize
comment mode, change thresholds, or approve a new model and retention policy.

**Clarify**: Name the decision-making roles and required approvals for pilot
selection, policy changes, model/provider changes, retention, threshold changes,
comment activation, incident response, and expansion.

## Editorial and SpecKit Metadata

- The feature branch is `TBD`; assign a branch identifier when implementation
  work is planned.
- The specification is marked Draft, but has no named owner or review/approval
  status. Add those fields if they are part of the team's SpecKit convention.
- The constitution and specification are dated but do not identify a linked
  amendment or specification version. Define how approved spec revisions are
  tracked against the constitution version.

## Overall Assessment

There is no broad conflict between the stated React/Vite, Node/Express, and
PostgreSQL implementation stack and the reviewed-repository language scope; the
specification explicitly distinguishes them. The principal implementation
blockers are the changed-line MUST/SHOULD mismatch, unresolved data and model
controls, missing configuration and publication contracts, and unmeasured
quality gates. Resolve C-001 through C-014 before enabling repository analysis;
resolve C-004, C-012, C-016, and C-022 before enabling inline comments or
expanding the pilot.
