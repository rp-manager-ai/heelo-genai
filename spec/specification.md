# Feature Specification: Project- and Domain-Specific PR Reviewer

**Feature Branch**: `TBD`
**Created**: 2026-10-02
**Status**: Draft
**Input**: `project_spec.md` and `spec/constitution.md`
**Spec Version**: 1.0.0
**Constitution Version**: 1.0.0
**Specification Owner**: Platform Engineering Lead (individual TBD)
**Approval Status**: Draft; pending required stakeholder review

## Summary

Build an AI-assisted pull request reviewer for a consumer-facing distributed
commerce platform and an engineering organization of approximately 100 people.
The service analyzes pull requests using code, repository policy, architecture
documents, and API contracts. It reports concise, evidence-backed risks to
authors and reviewers while leaving all review and merge decisions to people.

The prototype targets GitHub Enterprise Cloud and remains in shadow mode.
Inline comments require a separate future specification and manual approval.

## Users and Stakeholders

- **Pull request authors** need early, actionable feedback while developing a
  change.
- **Human reviewers** need focused risk signals and remain responsible for
  review decisions.
- **Platform and CI teams** install, configure, integrate, operate, and monitor
  the service.
- **Security and privacy owners** approve data handling, access, and activation
  controls.
- **Domain owners** define authoritative commerce invariants and adjudicate
  domain-specific findings.
- **Delivery leadership** evaluates pilot outcomes without using findings as
  individual performance measures.

## Scope

### In Scope

- GitHub Enterprise Cloud pull-request events and a non-blocking CI/check
  workflow.
- Repository-specific YAML policy, explicitly referenced architecture decision
  records (ADRs), and API schemas/contracts; dependency-graph expansion is out
  of scope.
- Risk analysis for security, privacy, distributed reliability, material
  maintainability, test coverage, and operability.
- Shadow evaluation through a GitHub Check Run summary.
- Review policy and evaluation of a single selected pilot repository.
- Auditing and evaluation of quality, latency, reliability, and developer
  feedback.

### Out of Scope

- Autonomously approving, rejecting, merging, or blocking pull requests.
- Editing source code, creating fix commits, or replacing human review, CI,
  static analysis, or security scanners.
- General-purpose code generation unrelated to pull-request review.
- Supporting every language or repository type in the initial release.
- Domain-specific order, refund, cancellation, payment, and event-invariant
  checks in the prototype.
- Inline PR comments, automatic comment-mode activation, artifacts, dashboards,
  and a user-facing React application in the prototype.
- Transitive dependency-graph expansion, mixed-language review, and handling of
  unsupported file types in the prototype.
- Durable queue replay, multi-repository expansion, quantitative SLO
  commitments, and advanced database recovery guarantees.

## User Scenarios & Testing

### User Story 1 - Secure Repository Enrollment (Priority: P1)

As a platform administrator, I can install the GitHub App only for selected
organizations and repositories with least-privilege access, so review jobs can
read authorized pull requests without exposing other repository data.

**Why this priority**: Analysis must not begin unless installation identity,
repository authorization, and webhook authenticity are trustworthy.

**Independent Test**: Install against a pilot repository, submit a valid signed
event, then submit invalid-signature and unauthorized-repository events. Verify
that only the valid authorized event can enqueue work or read source.

**Acceptance Scenarios**:

1. **Given** an authorized installation and repository, **When** a valid
   supported webhook arrives, **Then** the event is accepted and associated
   with that installation and repository.
2. **Given** an invalid webhook signature, **When** the event arrives,
   **Then** it is rejected and no review job is enqueued.
3. **Given** a repository outside the installation's authorized scope,
   **When** an event references it, **Then** no source is accessed and no review
   job is enqueued.

### User Story 2 - Review a Pull Request Revision (Priority: P1)

As a pull request author, I receive a review for a supported pull-request
revision that considers the full change and relevant dependencies, so material
risks are not missed by inspecting changed lines in isolation.

**Why this priority**: Revision-pinned, bounded context collection and reliable
trigger handling are prerequisites for trustworthy findings.

**Independent Test**: Send opened, reopened, synchronize, and ready-for-review
events for a test pull request. Verify revision pinning, retry deduplication,
superseded-revision handling, context limits, and configured path exclusions.

**Acceptance Scenarios**:

1. **Given** a supported pull-request event, **When** it is accepted,
   **Then** the review is pinned to the event's head commit SHA.
2. **Given** a retry for the same event and head SHA, **When** it is processed,
   **Then** it does not create a duplicate review or duplicate findings.
3. **Given** a newer head SHA supersedes an unfinished review, **When** the
   older job attempts publication, **Then** stale results are not published as
   findings for the newer revision.
4. **Given** configured exclusions or context limits, **When** context is
   collected, **Then** excluded files are omitted and the review reports any
   material partial or skipped coverage.

### User Story 3 - Receive Validated Findings (Priority: P1)

As an author or reviewer, I receive concise findings only when they identify a
concrete risk supported by changed code and trusted project context, so the
review is useful rather than speculative or noisy.

**Why this priority**: Evidence and location validation are essential to trust
and safe publication.

**Independent Test**: Evaluate known true-risk, clean, ambiguous, and malformed
model-output cases. Verify schema, rationale, severity, confidence, changed-line
location, duplicate suppression, and low-confidence filtering.

**Acceptance Scenarios**:

1. **Given** a candidate with concrete impact and supported changed-line
   evidence, **When** it passes schema and confidence checks, **Then** it is
   eligible for the configured result surface.
2. **Given** a candidate without a defensible location, rationale, severity,
   confidence, or supporting evidence, **When** validation runs, **Then** it is
   rejected or withheld.
3. **Given** uncertain evidence or unavailable context, **When** analysis ends,
   **Then** the result is marked partial or inconclusive and no unsupported
   finding is emitted.
4. **Given** two candidates describing the same issue, **When** findings are
   normalized, **Then** duplicates are suppressed or represented as one finding.

### User Story 4 - Operate in Shadow Mode (Priority: P1)

As a pilot team, we can inspect and label review results without public inline
comments or merge impact, so finding quality can be measured before comments
are enabled.

**Why this priority**: Shadow mode is the required quality and safety gate for
the rollout.

**Independent Test**: Run a labeled set of historical and synthetic pull
requests in shadow mode. Verify results appear only in the approved shadow
surface, do not block CI or merges, and capture evaluation metrics and feedback.

**Acceptance Scenarios**:

1. **Given** shadow mode is enabled, **When** a review completes, **Then** its
   result is recorded in the approved non-blocking CI/check surface and no
   inline PR comment is posted.
2. **Given** a GitHub or model outage, **When** retries are exhausted,
   **Then** the check reports an observable inconclusive outcome and does not
   fail or block merge.
3. **Given** a reviewer labels a finding, **When** evaluation data is recorded,
   **Then** it can be analyzed by category and severity without treating it as
   an individual performance measure.

### User Story 5 - Enable Controlled Inline Comments (Priority: P2)

**Prototype Status**: Out of scope. The behavior below is a future-release
contract only; the prototype remains in shadow mode.

As an authorized pilot owner, I can enable inline comments for selected
repositories only after agreed quality and security criteria are met, so useful
findings reach authors without flooding pull requests.

**Why this priority**: Inline publication depends on successful shadow
evaluation and explicit stakeholder approval.

**Independent Test**: Enable comment mode for one approved repository. Verify
only above-threshold actionable findings are posted on changed lines, reruns
update or resolve prior comments idempotently, and unauthorized repositories
remain in shadow mode.

**Acceptance Scenarios**:

1. **Given** comment mode is enabled for an approved repository, **When** a
   finding passes publication checks, **Then** a concise inline comment is
   anchored to a relevant changed line.
2. **Given** a finding falls below the configured confidence threshold,
   **When** publication is attempted, **Then** no inline comment is posted.
3. **Given** the same review is rerun, **When** prior service comments exist,
   **Then** comments are updated, resolved, or left unchanged without creating
   duplicates.
4. **Given** a repository is not approved for comment mode, **When** it is
   reviewed, **Then** inline comments are not posted.

### User Story 6 - Configure and Audit Reviews (Priority: P1)

As a platform or repository administrator, I can validate versioned review
policy and inspect an audit trail, so behavior is explainable, reproducible,
and constrained by organization controls.

**Why this priority**: Invalid policy, untraceable changes, or unsafe overrides
can undermine security and review consistency.

**Independent Test**: Validate valid, malformed, missing, and policy-violating
repository configurations. Inspect audit records for configuration changes and
reviews, confirming required metadata is present and raw source is absent.

**Acceptance Scenarios**:

1. **Given** a valid repository YAML configuration, **When** it is loaded,
   **Then** it is schema-validated and its version is attached to the review.
2. **Given** missing or malformed configuration, **When** a review is started,
   **Then** a clear diagnostic is recorded and safe organization defaults are
   applied.
3. **Given** repository configuration attempts to disable a mandatory control
   or select an unapproved model, **When** validation runs, **Then** the
   override is rejected.
4. **Given** a review or configuration change, **When** its audit event is
   stored, **Then** the required metadata is available to authorized operators
   without raw source, secrets, or unnecessary PII.

## Functional Requirements

- **FR-001**: The service MUST support GitHub Enterprise Cloud installations
  scoped to selected organizations and repositories.
- **FR-002**: The service MUST validate webhook signatures and authorize every
  event and GitHub operation against its installation and repository.
- **FR-003**: The service MUST process pull-request opened, reopened,
  synchronize, and ready-for-review events.
- **FR-004**: Each review MUST be pinned to a pull-request head SHA. Retries and
  superseded revisions MUST NOT publish duplicate or stale findings.
- **FR-005**: The service MUST retrieve changed files and explicitly referenced
  same-repository policy, ADR, and API schema/contract documents.
- **FR-005a**: The prototype MUST NOT perform transitive dependency-graph
  expansion. Context MUST be revision-pinned and bounded.
- **FR-006**: Context collection MUST respect configured excluded paths and
  explicit limits for file size, file count, diff size, and model context.
- **FR-007**: The service MUST identify changed components and applicable
  policies before analysis. Domain rules MUST only be applied when supported by
  repository guidance or available evidence.
- **FR-008**: The supported changed-file scope MUST be TypeScript/Node.js and
  Terraform, Kubernetes, and Helm definitions in one selected pilot repository.
- **FR-009**: Domain-specific commerce checks for orders, refunds,
  cancellations, payment, and event invariants are out of scope for the
  prototype.
- **FR-010**: The service MUST use only an approved enterprise model hosted in
  company cloud or private network infrastructure. Repository content MUST NOT
  be sent to public model endpoints.
- **FR-011**: The service MUST redact secrets and configured sensitive data
  before model requests and send only the minimum context required.
- **FR-012**: Repository code, comments, documentation, and generated artifacts
  MUST be treated as untrusted input, not instructions. Model output MUST NOT
  call tools or expand its permissions.
- **FR-013**: Model analysis MUST use structured input and output. Candidate
  findings MUST be schema-validated before publication.
- **FR-014**: A publishable finding MUST include a stable key, repository,
  pull request, commit SHA, file, changed-line range, category, severity,
  confidence, concise title and rationale, evidence references, and optional
  remediation or focused test.
- **FR-015**: Every publishable finding MUST anchor to one or more changed
  lines. Findings supported only by unchanged context MUST be withheld.
- **FR-016**: The prototype MUST use a non-blocking GitHub Check Run summary
  only. It MUST NOT publish inline comments, artifacts, or dashboards.
- **FR-017**: Inline comment publication is out of scope for the prototype.
  Any future comment mode MUST require manual owner approval, an approved
  confidence threshold, changed-line anchors, and idempotent update/resolution.
- **FR-018**: The service MUST provide repository opt-out and feedback controls
  before expanding the comment pilot.
- **FR-019**: Transient GitHub and model failures MUST use bounded retries and
  backoff. Exhausted retries or unavailable context MUST produce an observable,
  partial or inconclusive, non-blocking result.
- **FR-020**: Review failure MUST NOT fail a required CI check, block merges, or
  prevent normal delivery in the initial release.
- **FR-021**: The pilot repository MAY provide `.pr-reviewer.yml`, schema
  version 1, validated against the published JSON Schema.
- **FR-022**: Missing or invalid repository configuration MUST produce a clear
  diagnostic and use valid organization defaults. If no valid organization
  defaults exist, the review MUST be skipped non-blockingly.
- **FR-023**: Repository configuration MUST NOT disable mandatory organization
  security controls or select an unapproved model endpoint.
- **FR-024**: The service MUST record review ID, repository, pull request,
  revision, triggering event, configuration version, model/version, outcome,
  timing, and check/comment identifiers as applicable.
- **FR-025**: Operational logs and persistent records MUST exclude raw source,
  prompts, secrets, and unnecessary PII. Audit access MUST be restricted.
  Approved retention and deletion periods MUST be configured before the pilot.
- **FR-026**: The system MUST support evaluation using labeled historical and
  synthetic pull requests, including clean changes and true-risk cases.

## Non-Functional Requirements

- **NFR-001 Security**: Encrypt data in transit and at rest, isolate jobs by
  organization/repository, and use managed secret storage for credentials.
- **NFR-002 Privacy**: Define and test redaction coverage and fail-safe behavior
  before any model request. Keep source content out of persistent logs by
  default.
- **NFR-003 Reliability**: Bound concurrency, execution time, retries, diff
  size, file count, and context size; identify partial or skipped reviews.
- **NFR-004 Auditability**: Record source revision, configuration version,
  prompt/rule version, and model version so review behavior can be explained.
- **NFR-005 Scale**: Support a pilot organization of approximately 100 people
  and bursty pull-request traffic through queue-based horizontal scaling.
- **NFR-006 Performance**: The prototype MUST record review duration and queue
  delay for evaluation. No p95 completion-time commitment is in scope.
- **NFR-007 Availability**: The prototype MUST expose health and failure
  status for operators. No monthly availability commitment is in scope.
- **NFR-008 Delivery safety**: The reviewer MUST remain advisory and
  non-blocking in the initial release, including during degraded operation.

## Technology Constraints

- The reviewer web frontend MUST use React 18 with Vite.
- The service backend MUST use Node.js with Express.
- Persistent application data MUST use PostgreSQL 15.
- PostgreSQL MUST run through Docker in local development and in deployment
  environments that use containers.
- The reviewed-repository language scope in FR-008 is separate from the
  implementation stack of this reviewer service.

## Key Entities

- **Installation**: GitHub App identity, authorized organization/repositories,
  permissions, and installation status.
- **Pull Request Revision**: Repository, pull-request number, head SHA, base
  SHA, event, and review status.
- **Review Job**: Revision-pinned unit of work with deduplication identity,
  bounded execution state, retries, and outcome.
- **Repository Policy**: Versioned YAML guidance, enabled categories, path
  rules, domain invariants, data classifications, thresholds, and mode settings.
- **Context Source**: Diff/file content or approved guidance, ADR, API contract,
  and dependency reference used for analysis.
- **Finding**: Stable key, changed-line location, category, severity,
  confidence, rationale, evidence, and optional remediation.
- **Publication**: Shadow check/artifact or inline comment state associated with
  a validated finding and review revision.
- **Audit Event**: Review or configuration event with actor/source identity,
  revision/version, model, outcome, timing, and related GitHub identifiers.
- **Evaluation Label**: Reviewer feedback used to assess quality by finding
  category and severity.

## Success Criteria

- **SC-001**: All tested invalid-signature and unauthorized-repository events
  are rejected before source access or job enqueue.
- **SC-002**: Repeated delivery of the same event and head SHA creates no
  duplicate review publication; stale-head findings are not published.
- **SC-003**: Tests with seeded secrets demonstrate that secrets and configured
  sensitive values are absent from outbound model payloads and ordinary logs.
- **SC-004**: Only the approved private enterprise model endpoint receives
  repository context in security tests.
- **SC-005**: Every published finding passes schema, evidence, confidence, and
  changed-line validation; invalid candidates are withheld.
- **SC-006**: Shadow-mode tests confirm no inline comments and no merge/CI
  blocking, including exhausted-retry and provider-outage cases.
- **SC-007**: The prototype posts no inline comments. Repeated delivery of the
  same event does not duplicate its GitHub Check Run.
- **SC-008**: Pilot reports precision/actionability, false-positive share,
  acceptance/dismissal feedback, valid-line anchoring, latency, reliability,
  and coverage by category and severity.
- **SC-009**: The prototype remains in shadow mode. Security and privacy owners
  approve its configuration and data handling before repository analysis starts.
- **SC-010**: Review duration, queue delay, completion state, and provider
  failures are measured; no latency or availability SLO is claimed.

## Assumptions

- GitHub Enterprise Cloud supports the App, webhook, check, and comment
  permissions required by the selected pilot configuration.
- An approved private enterprise model and managed secret service are available
  before reviews process repository content.
- The selected pilot repository can provide a maintainer for YAML policy and
  feedback; commerce-specific domain rules are not enabled in the prototype.
- ADRs, API contracts, and other approved repository context are accessible at
  the reviewed revision or are explicitly versioned references.
- Prototype jobs use a best-effort in-process queue; restart-safe replay and
  durable queue guarantees are out of scope.

## Dependencies

- GitHub Enterprise Cloud App registration, installation, webhook, and Check
  Run capabilities.
- Approved enterprise model endpoint, model governance, and data-processing
  terms.
- Managed secrets service and approved hosting/network boundary.
- Repository YAML schema, organization defaults, and configuration ownership.
- Domain-owner decisions are required only before a future release adds
  commerce-specific checks.
- Labeled historical and synthetic pull requests and reviewer feedback process.
- CI integration and pilot agreement on the shadow result surface.

## Risks and Open Decisions

- Select the approved model/provider, deployment endpoint, and data-processing
  terms before repository content is sent for inference.
- Security and privacy owners must approve the retention period and data
  residency before the pilot; source, prompts, and full traces are not stored.
- Select the one pilot repository and verify GitHub App permissions in its
  GitHub Enterprise Cloud organization.
- Supply valid organization defaults for repository policy before analysis.
- Identify individuals for the specified Platform, Security, and Privacy owner
  roles before pilot activation.
- Additional context sources, commerce-specific rules, broader languages,
  multi-repository rollout, SLO commitments, and comment activation are
  deferred or out of scope as specified below.

## Rollout

1. **Discovery and policy**: Select one pilot repository; confirm the approved
  model, privacy/retention rules, evaluation labels, and organization policy.
2. **Foundation**: Implement GitHub App authentication, event processing,
   queue/workers, YAML validation, redaction, audit events, and non-blocking
   result reporting.
3. **Shadow pilot**: Evaluate the selected repository without inline comments;
  collect labels and measure general-risk finding quality.
4. **Future release**: Inline comments, additional repositories, and expansion
  require a separate approved specification and rollout decision.

## Initial Pilot Acceptance Criteria

- Webhook authenticity and installation/repository authorization are verified.
- Reviews are pinned to a head SHA, deduplicated, and bounded by configured
  limits.
- YAML configuration is schema-validated, versioned, and fails safely with a
  clear diagnostic.
- Redaction precedes model calls; seeded-secret tests confirm no leakage to
  model requests or ordinary logs.
- Only the approved private model endpoint is used.
- Findings pass schema, evidence, confidence, and changed-line validation.
- Shadow mode posts no inline comments and does not block merges. Comment mode
  is not available in the prototype.
- Failures are observable and non-blocking; unsupported findings are never
  fabricated when context or inference is unavailable.
- Evaluation reports quality, false positives, valid-line anchoring, latency,
  reliability, and reviewer feedback by category.
- Security and privacy owners approve pilot configuration and data handling.
  Domain-owner approval is required only before adding domain-specific rules.

## Confirmed Prototype Decisions

The following dispositions were confirmed by the project owner. This section
controls wherever an earlier draft statement conflicts with it.

### Decisions Added to the Specification

#### C-001 - Changed-Line Evidence

Every publishable finding MUST anchor to one or more changed lines. A
multi-line finding may span only relevant changed lines. Unchanged context may
support the rationale but cannot be the finding's sole anchor. Unsupported
findings are withheld.

#### C-003 - Finding Contract

The finding schema is JSON Schema version 1. Categories are `security`,
`privacy`, `reliability`, `maintainability`, `testing`, and `operability`.
Severity is `critical`, `high`, `medium`, or `low`. Confidence is a numeric
value from 0.0 through 1.0. Evidence references identify a repository path,
revision, and line range or policy/contract path and version. A stable finding
key is based on repository, pull request, file, category, changed-line anchor,
and normalized title. A finding may omit remediation when no clear fix exists.

#### C-006 - Repository Configuration

The configuration path is `.pr-reviewer.yml`, with `schemaVersion: 1`. The
published JSON Schema defines enabled categories, excluded/high-risk paths,
stack hints, data classifications, redaction hints, and repository guidance
references. Organization policy is authoritative; repository values may
narrow scope but cannot weaken mandatory controls or select a model, threshold,
or publication mode. Missing/invalid repository configuration uses valid
organization defaults. If those defaults are absent or invalid, the review is
skipped and the check reports `configuration_unavailable` without blocking CI.

#### C-008 - Pull-Request Lifecycle

The prototype starts reviews for non-draft `opened`, `reopened`, `synchronize`,
and `ready_for_review` events. Draft PRs are ignored until ready. A `closed`
event cancels pending work; a merged PR is treated as closed. A publication
guard MUST compare the job head SHA with the current PR head before writing a
result. Webhook delivery ID plus repository, PR number, and head SHA form the
deduplication identity. Manual reruns are out of scope.

#### C-010 - GitHub Access

The pilot uses one GitHub App installation scoped to one selected repository.
The App reads pull requests and repository contents and writes GitHub Check
Runs. It receives no comment-write permission in the prototype. A platform
administrator owns installation and mode configuration. Installation tokens
are short-lived and credentials are held in managed secret storage. API
rate-limit responses are respected; retries remain bounded.

#### C-012 - Data Lifecycle

PostgreSQL stores review state, finding summaries, non-sensitive evidence
references, configuration/model/rule versions, check identifiers, and audit
metadata. Raw diffs, source files, prompts, and full model traces are transient
and MUST NOT be persisted. A retention period for findings and audit metadata
must be approved by security/privacy owners and configured before the pilot;
the pilot MUST NOT start without it. Repository removal triggers deletion under
that approved policy.

#### C-014 - Publication Behavior

The prototype publishes only a Check Run summary. A completed review with or
without findings has a successful, non-required check; partial, skipped,
inconclusive, or failed outcomes use a neutral, non-required check with an
explicit state in the summary. Future inline comments, if separately approved, use one comment
per stable finding key, update the existing comment when the finding persists,
and resolve the service's prior comment when it is absent from a newer
completed revision. Comment mode never activates automatically.

#### C-016 - Evaluation Protocol

The evaluation set includes historical and synthetic pull requests, both
known-risk and clean changes, with results segmented by category and severity.
Two reviewers label each evaluated case independently; disagreements are
adjudicated by a designated senior reviewer. Tuning examples and evaluation
examples are kept separate by pull request. Precision is the share of emitted
findings labeled actionable; false-positive share is the share of emitted
findings labeled unsupported; valid-line anchoring is the share of findings
mapped to changed lines. Sample counts and denominators are always reported.
No minimum sample size or numeric promotion threshold is promised; results do
not enable comment mode.

#### C-018 - Operational States

Each review has one terminal state: `completed`, `partial`, `skipped`,
`inconclusive`, or `failed`. A completed review with no findings is distinct
from every non-completed state. Structured logs MUST include review ID, PR
revision, stage, duration, and error class, but not source, prompts, secrets,
or finding text. Basic health status and counters for queue delay, duration,
completion state, retry, and provider failure are required. Platform
Engineering owns operational triage. A service-specific dashboard and
24-hour response commitment are out of scope.

#### C-020 - Internal Contracts

The webhook/API contract is documented with OpenAPI 3.1. Repository
configuration and finding payloads use versioned JSON Schema. Internal worker
messages carry schema versions and the pinned PR head SHA. No externally
supported API compatibility guarantee is part of the prototype; breaking
internal contract changes require coordinated API/worker deployment.

#### C-022 - Approval Roles

The Platform Engineering Lead owns pilot selection and operations. A Security
Owner approves GitHub permissions, model endpoint, and threat controls. A
Privacy Owner approves data handling and retention. The repository maintainer
approves repository configuration. These role approvals are required before
pilot analysis begins. Commerce-domain approval is unnecessary while
domain-specific checks are out of scope. Comment activation requires a separate
future approval and is not available in this prototype.

#### C-024 - Specification Owner and Status

The specification owner is the Platform Engineering Lead; the individual is
TBD. Status remains Draft until the required platform, security, and privacy
reviews are recorded.

#### C-025 - Document Version Linkage

This specification is version 1.0.0 and was reviewed against Constitution
version 1.0.0. Every approved specification revision records the constitution
version it implements. A constitution amendment that changes a requirement
requires review of this specification before implementation continues.

### Confirmed Out-of-Scope Items

#### C-002 - Rich Result Surfaces

Artifacts and dashboards are out of scope. The GitHub Check Run summary is the
only prototype result surface and follows the status behavior defined under
C-014.

#### C-004 - Automated Quality Gate

Automated threshold-based comment activation is out of scope. The prototype
remains in shadow mode and does not publish inline comments. Evaluation results
are advisory; any later comment activation requires manual approval and an
approved threshold.

#### C-005 - Commerce-Specific Rules

Order, refund, cancellation, payment, and event-invariant checks are out of
scope. The prototype performs only the general risk categories in FR-007.

#### C-007 - Dependency Expansion and Unsupported Inputs

Transitive dependency traversal, mixed-language review, generated/binary/vendor
file analysis, and unsupported-file fallback behavior are out of scope. Only
supported changed files and explicitly referenced same-repository context are
analyzed. Pilot enrollment is limited to a repository whose changes fit the
supported-file scope.

#### C-009 - Durable Queue Guarantees

Restart-safe queue durability, dead-letter storage, replay, and per-repository
fairness are out of scope. The in-process queue is best effort; a process
restart may lose pending work. Retries are limited to one retry for a
transient GitHub or model error, with bounded backoff.

#### C-011 - Extended Detection and Red-Team Program

Custom detection engines and a separate red-team program are out of scope.
Mandatory baseline secret/configured-sensitive-data redaction, fail-closed
behavior on redaction failure, untrusted-input treatment, and disabled model
tool use remain required by the Constitution and FR-010 through FR-012.

#### C-013 - Human Identity Audit Fields

Individual human identity fields are out of scope for application audit data.
The service records installation/repository identity and configuration
version. Human administrator activity remains available through GitHub or
organization audit systems, not copied into application records.

#### C-015 - User-Facing Frontend

Operator consoles, dashboards, configuration editors, and other user-facing
React screens are out of scope. The React 18/Vite constraint applies if a
frontend is added in a future release.

#### C-017 - Committed SLOs and Capacity Guarantees

The provisional 5-minute p95 and 99.5% monthly availability targets are not
prototype commitments. The prototype measures duration and availability for
later capacity planning but makes no SLO guarantee.

#### C-019 - Advanced Persistence Recovery

Backup/restore objectives, point-in-time recovery, and restart-safe queue
recovery are out of scope. Minimal PostgreSQL persistence required by FR-024
remains in scope; no raw source or prompt content is stored.

#### C-021 - Multi-Repository and Mixed-Language Expansion

The prototype is limited to one selected pilot repository and the supported
file scope in FR-008. Multi-repository expansion and mixed-language analysis
require a separate specification and approval.

#### C-023 - Feature Branch Metadata

The feature branch remains `TBD`; branch assignment is deferred until
implementation planning.
