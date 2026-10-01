# Implementation Tasks: Project- and Domain-Specific PR Reviewer

**Plan**: `spec/plan.md`
**Specification**: `spec/specification.md` v1.0.0
**Status**: Draft
**Task order**: Complete tasks in ID order unless a dependency is explicitly
listed.

## Task Conventions

- Tasks are independently reviewable and use unchecked boxes until completed.
- A task is complete only when all listed acceptance criteria pass and its
  tests or approval evidence are recorded.
- `[Owner]` identifies a role, not a named individual; M0 must record the
  person assigned to each role.
- Numeric bounds, endpoint names, and retention periods are supplied by
  approved pilot configuration, not invented in implementation tasks.

## Phase 0: Readiness and Scope Freeze

### T001 - Select Pilot Repository

**Owner**: Platform Engineering Lead
**Depends on**: None
**Execution Status**: Blocked pending project-owner confirmation.

**Assessment**: The current workspace is `heelo-genai`, but its Git `origin`
points to `github.com/raviepam/jira-confluence-automation.git`. This does not
confirm the intended pilot repository or its GitHub Enterprise Cloud status.
No repository-maintainer scope confirmation is available. Do not mark this task
complete or treat the current remote as the pilot without owner confirmation.

- [x] Select one GitHub Enterprise Cloud repository for the prototype.
- [x] Confirm its expected changes fit TypeScript/Node.js and Terraform,
  Kubernetes, or Helm changed-file scope.

**Execution Status**: Complete by project-owner confirmation.

**Recorded Selection**:

- Repository: `raviepam/jira-confluence-automation`
- GitHub Enterprise Cloud: confirmed by the project owner.
- Repository maintainer: RP.
- GitHub App installation owner: RP.
- Supported file-scope fit: confirmed by the project owner and maintainer.

**Acceptance Criteria**:

- A repository identifier and installation owner are recorded.
- The repository maintainer confirms the expected file scope.
- Multi-repository expansion and unsupported/mixed-language review are not
  included in this pilot.

### T002 - Assign Pilot Approval Roles

**Owner**: Platform Engineering Lead
**Depends on**: None

- [ ] Name the Platform Engineering Lead, Security Owner, Privacy Owner, and
  pilot repository maintainer.
- [ ] Record each role's approval and incident responsibilities.

**Acceptance Criteria**:

- Each required role has a named person and contact path.
- The Security Owner, Privacy Owner, and repository maintainer accept their
  responsibilities before source processing is enabled.

### T003 - Approve Private Model Boundary

**Owner**: Security Owner
**Depends on**: T002

- [ ] Select the approved enterprise model provider, endpoint, and processing
  region.
- [ ] Record approved data-processing terms and network allowlist requirements.

**Acceptance Criteria**:

- The exact approved endpoint and region are recorded in protected
  environment configuration.
- Model terms permit the planned repository-content processing.
- No source-bearing model request can run before this approval is recorded.

### T004 - Approve Data Retention and Residency

**Owner**: Privacy Owner
**Depends on**: T002

- [ ] Set retention and deletion periods for finding summaries and audit
  metadata.
- [ ] Define data residency and repository-removal behavior.

**Acceptance Criteria**:

- Retention periods and residency are recorded for each persisted data class.
- Repository removal has an approved deletion outcome.
- Pilot activation is blocked until this decision is approved and configurable.

### T005 - Provide Organization Policy Defaults

**Owner**: Platform Engineering Lead
**Depends on**: T001, T002

- [ ] Provide valid organization defaults for enabled general-risk categories,
  path controls, data classifications, and approved context references.
- [ ] Identify the maintainer and version for the pilot repository's
  `.pr-reviewer.yml`.

**Acceptance Criteria**:

- The defaults and repository policy are versioned and have an assigned owner.
- The repository configuration cannot select a model or weaken organization
  security controls.
- Missing/invalid defaults are understood to skip review non-blockingly.

### T006 - Approve Evaluation Set and Shadow Audience

**Owner**: Platform Engineering Lead
**Depends on**: T001, T002

- [ ] Select historical and synthetic pull requests containing known-risk and
  clean examples.
- [ ] Name two independent reviewers and a senior adjudicator.
- [ ] Confirm who can see the GitHub Check Run summary.

**Acceptance Criteria**:

- The corpus has documented sources, labels, categories, and severity coverage.
- Tuning examples and evaluation examples are separated by pull request.
- The intended Check Run audience and label process are recorded.

### T007 - Set Single-Process Capacity Boundary

**Owner**: Platform Engineering Lead
**Depends on**: T001

- [ ] Reconcile NFR-005's horizontal-scaling language with the prototype's
  best-effort in-process queue.
- [ ] Record acceptable pilot volume, maximum active work, and overload
  behavior.

**Acceptance Criteria**:

- Pilot volume and active-work limits have explicit approved values.
- The single-process/restart-loss limitation is accepted by the pilot owner.
- If horizontal scaling is required, implementation is blocked pending a
  specification and plan revision.

**Milestone M0 Gate**: T001-T007 are complete and approvals are recorded before
any source-processing implementation begins.

## Phase 1: Runtime, Contracts, and Local Data Foundation

### T008 - Define Workspace and Module Boundaries

**Owner**: Platform Engineering
**Depends on**: T007

- [ ] Wire `apps/api`, `apps/worker`, and the existing `packages/*` modules
  into reproducible root scripts and package boundaries.
- [ ] Preserve the repository's ESM JavaScript conventions unless a tooling
  change is separately approved.

**Acceptance Criteria**:

- A clean install resolves every declared workspace/package dependency.
- API, worker, config, GitHub, model, review-core, and shared code have clear
  import boundaries.
- No user-facing frontend feature is introduced by this task.

### T009 - Pin Runtime and Preserve Existing Tests

**Owner**: Platform Engineering
**Depends on**: T008

- [ ] Document and pin the supported Node.js runtime and package manager.
- [ ] Keep the existing root health/chat tests passing or document an approved
  migration of their behavior.

**Acceptance Criteria**:

- The documented runtime is used by local development and CI.
- A clean checkout can run the documented install and test commands.
- Existing supported routes have regression coverage or an approved removal
  decision.

### T010 - Add Local PostgreSQL 15 Service

**Owner**: Platform Engineering
**Depends on**: T008

- [ ] Add Docker-based PostgreSQL 15 configuration and environment-driven
  connection settings.
- [ ] Add a database readiness check without placing credentials in source
  control.

**Acceptance Criteria**:

- The documented Docker command starts PostgreSQL 15 locally.
- The application can verify readiness using non-committed environment
  configuration.
- No database credential is committed to the repository.

### T011 - Add Migrations and Persistence Constraints

**Owner**: Platform Engineering
**Depends on**: T010, T004

- [ ] Add a migration runner and initial tables for review state, finding
  summaries, Check Run state, configuration versions, and approved audit data.
- [ ] Define transaction and unique-key constraints for review idempotency.

**Acceptance Criteria**:

- Migrations apply to an empty PostgreSQL 15 database and report the schema
  version.
- Duplicate review identities cannot create duplicate persisted review state.
- Stored fields exclude raw source, diffs, prompts, full traces, and human
  identity fields.

### T012 - Define Versioned Service Contracts

**Owner**: Platform Engineering
**Depends on**: T008

- [ ] Add OpenAPI 3.1 for the webhook/API surface.
- [ ] Add JSON Schema version 1 for repository config, findings, worker
  messages, and review outcomes.

**Acceptance Criteria**:

- Each contract has an explicit version and validation command/test.
- Worker messages include the pinned PR head SHA and schema version.
- Invalid payloads fail validation before model calls or persistence.

### T013 - Establish CI Quality Gates

**Owner**: Platform Engineering
**Depends on**: T009, T011, T012

- [ ] Add CI checks for tests, schema validation, migrations, and markdown
  validation.
- [ ] Keep the reviewer Check Run optional and non-required.

**Acceptance Criteria**:

- CI runs the documented checks on a clean checkout.
- A failing validation/test fails its development CI job with an actionable
  diagnostic.
- Branch protection does not require the reviewer Check Run.

**Milestone M1 Gate**: T008-T013 pass from a clean checkout, including local
PostgreSQL startup, migrations, schemas, and tests.

## Phase 2: Authenticated GitHub Intake and Revision Safety

### T014 - Implement Signed Webhook Endpoint

**Owner**: API Engineer
**Depends on**: T002, T008, T012

- [ ] Implement the Express webhook endpoint using raw request bytes for
  signature validation.
- [ ] Authenticate installation and authorize the repository before reading
  content or enqueueing work.

**Acceptance Criteria**:

- Valid signed events from the selected installation are accepted.
- Invalid signatures are rejected without source access or job creation.
- Events for unauthorized repositories are rejected without source access or
  job creation.

### T015 - Implement Event and Draft Lifecycle

**Owner**: API Engineer
**Depends on**: T014

- [ ] Handle non-draft `opened`, `reopened`, `synchronize`, and
  `ready_for_review` events.
- [ ] Ignore draft PRs until ready and cancel pending work on close/merge where
  possible.

**Acceptance Criteria**:

- Each supported event maps to a documented review transition.
- Draft PR events do not start analysis before ready-for-review.
- Close/merge events prevent pending work from publishing when cancellation
  is possible.

### T016 - Persist Event Identity and Deduplicate

**Owner**: API Engineer
**Depends on**: T011, T014

- [ ] Persist webhook delivery ID, repository, PR number, head SHA, and review
  state before enqueueing accepted work.

**Acceptance Criteria**:

- Re-delivery with the same identity does not enqueue duplicate work.
- Database uniqueness and transaction tests cover concurrent duplicate
  deliveries.
- Deduplication state contains no source content.

### T017 - Add Bounded In-Process Queue

**Owner**: Worker Engineer
**Depends on**: T007, T008, T016

- [ ] Enqueue accepted review jobs in the in-process worker queue.
- [ ] Enforce the approved active-work limit and return a non-blocking webhook
  acknowledgement.

**Acceptance Criteria**:

- Queue saturation follows the approved overload behavior from T007.
- A process restart can lose pending jobs and this limitation is documented.
- No claim of durable delivery, replay, fairness, or horizontal scale is made.

### T018 - Guard Against Stale Head Publication

**Owner**: Worker Engineer
**Depends on**: T015, T016

- [ ] Verify the current PR head SHA before any result publication.
- [ ] Suppress results for a job whose revision has been superseded.

**Acceptance Criteria**:

- A matching head SHA can proceed to publication.
- A stale head SHA produces no Check Run update for the newer revision and is
  recorded as superseded.
- Tests cover a head update during active analysis.

### T019 - Implement Bounded Retry Policy

**Owner**: API/Worker Engineers
**Depends on**: T014, T017

- [ ] Retry transient GitHub/model errors at most once with bounded backoff.
- [ ] Do not retry permanent authorization, configuration, or validation
  failures.

**Acceptance Criteria**:

- Tests prove the one-retry limit and terminal outcome after exhaustion.
- Permanent failures do not consume the transient retry budget.
- Retry metadata is recorded without logging secrets or source.

**Milestone M2 Gate**: T014-T019 pass signature, authorization, deduplication,
stale-head, draft lifecycle, and bounded-work tests.

## Phase 3: Repository Policy and Bounded Context

### T020 - Validate Repository YAML

**Owner**: Configuration Engineer
**Depends on**: T005, T012

- [ ] Load `.pr-reviewer.yml` from the reviewed revision and validate schema
  version 1.
- [ ] Enforce organization policy over repository-supplied values.

**Acceptance Criteria**:

- Valid config produces a normalized policy object and records its version.
- Invalid config produces a structured diagnostic.
- Repository config cannot select a model, confidence threshold, or publication
  mode, and cannot weaken organization controls.

### T021 - Implement Safe Defaults and Skip Behavior

**Owner**: Configuration Engineer
**Depends on**: T020

- [ ] Apply valid organization defaults when repository config is absent or
  invalid.
- [ ] Skip review with `configuration_unavailable` if organization defaults
  are also invalid or absent.

**Acceptance Criteria**:

- Tests cover valid repo config, invalid/missing repo config with valid
  defaults, and invalid/missing defaults.
- The skip state is non-blocking and cannot invoke the model.
- The applied config/default version is attached to review metadata.

### T022 - Collect Revision-Pinned Changed Files

**Owner**: Context Engineer
**Depends on**: T014, T018, T020

- [ ] Retrieve the PR diff and supported changed files at the exact head SHA.
- [ ] Reject or skip retrieval when the requested revision cannot be verified.

**Acceptance Criteria**:

- Every collected file is associated with the job head SHA.
- A mismatched or unavailable revision produces a partial/inconclusive result,
  not substituted content.
- No transitive dependency graph is traversed.

### T023 - Load Explicit Context and Apply Exclusions

**Owner**: Context Engineer
**Depends on**: T020, T022

- [ ] Load only explicitly referenced same-repository policy, ADR, and API
  schema/contract documents.
- [ ] Apply configured excluded-path rules to changed files and references.

**Acceptance Criteria**:

- All context references are resolved at the reviewed revision or are rejected
  as unversioned/unavailable.
- Excluded paths are absent from model context.
- Unreferenced documents and dependency-graph content are not collected.

### T024 - Enforce Context Bounds and Coverage States

**Owner**: Context Engineer
**Depends on**: T007, T022, T023

- [ ] Enforce approved limits for file size, file count, diff size, and model
  context.
- [ ] Mark review coverage partial or skipped when the defined limits prevent
  supported analysis.

**Acceptance Criteria**:

- Limits come from approved configuration and are recorded with the review.
- Boundary tests cover values below, at, and above each configured limit.
- Partial/skipped outcomes explain omitted coverage without exposing source.

### T025 - Classify General Review Categories

**Owner**: Review-Core Engineer
**Depends on**: T020, T022

- [ ] Classify changed components and enabled general-risk categories.
- [ ] Keep order/refund/cancellation/payment/event-invariant checks disabled.

**Acceptance Criteria**:

- Enabled categories are limited to security, privacy, reliability,
  maintainability, testing, and operability.
- Category selection is recorded with the applied policy version.
- No commerce-specific rule is run in prototype tests.

### T026 - Produce Non-Sensitive Context Manifest

**Owner**: Context Engineer
**Depends on**: T023, T024

- [ ] Produce a manifest of included/excluded references and coverage state.
- [ ] Keep raw file contents out of the manifest and persistent storage.

**Acceptance Criteria**:

- The manifest can explain which source classes were considered without
  including source text or secrets.
- Tests verify exclusions and coverage state are reflected correctly.

**Milestone M3 Gate**: T020-T026 pass config precedence, revision pinning,
exclusion, context bounds, and partial/skipped coverage tests.

## Phase 4: Redaction, Model Analysis, and Finding Validation

### T027 - Redact Sensitive Values Before Inference

**Owner**: Security Engineer
**Depends on**: T003, T023, T024

- [ ] Apply baseline secret and configured-sensitive-data redaction before
  serialization to the model adapter.
- [ ] Fail closed if redaction fails.

**Acceptance Criteria**:

- Seeded secrets and configured sensitive values are absent from outbound test
  payloads.
- A redaction error prevents any model request and produces an observable,
  non-blocking terminal outcome.
- Redaction logs contain metadata only, never original sensitive values.

### T028 - Add Approved Private Model Adapter

**Owner**: Model Engineer
**Depends on**: T003, T012, T027

- [ ] Implement structured model requests against only the approved private
  endpoint.
- [ ] Enforce request-size and timeout bounds; disable model tool execution.

**Acceptance Criteria**:

- Endpoint configuration is allowlisted and comes from protected environment
  configuration.
- An unapproved endpoint is rejected before sending repository context.
- Request/response schemas and timeout/error behavior have automated tests.

### T029 - Test Untrusted Input and Model Failures

**Owner**: Security/Model Engineers
**Depends on**: T028

- [ ] Treat source, comments, documentation, generated files, and model output
  as untrusted data.
- [ ] Add prompt-injection, malformed response, timeout, and provider-failure
  fixtures.

**Acceptance Criteria**:

- Repository text cannot change model/system policy or activate tools.
- Malformed or unavailable model output creates no publishable finding.
- Failure outcome follows the bounded retry policy and does not block CI.

### T030 - Validate Finding Schema and Required Fields

**Owner**: Review-Core Engineer
**Depends on**: T012, T028

- [ ] Validate findings against JSON Schema version 1.
- [ ] Enforce category/severity enums, confidence range, rationale, impact,
  evidence, and required identifiers.

**Acceptance Criteria**:

- Valid findings pass schema validation and invalid findings are rejected.
- Confidence is numeric and between 0.0 and 1.0 inclusive.
- Severity and category values match the approved enums.

### T031 - Map Findings to Changed Lines and Deduplicate

**Owner**: Review-Core Engineer
**Depends on**: T022, T030

- [ ] Map each publishable finding to one or more relevant changed lines.
- [ ] Generate stable finding keys and suppress duplicate/unsupported findings.

**Acceptance Criteria**:

- Findings anchored only to unchanged context are withheld.
- Multi-line findings include only relevant changed lines.
- Equivalent findings produce the same stable key for the same revision and do
  not create duplicate output.

### T032 - Scan Output Before Storage and Publication

**Owner**: Security/Review-Core Engineers
**Depends on**: T027, T030

- [ ] Scan validated output for secrets and configured sensitive values.
- [ ] Persist and publish only approved summaries and non-sensitive references.

**Acceptance Criteria**:

- Seeded-secret tests find no sensitive values in database records, logs, or
  Check Run output.
- A finding containing a detected secret is withheld or safely sanitized per
  approved policy.
- Raw diffs, source files, prompts, and full traces are not persisted.

**Milestone M4 Gate**: T027-T032 pass redaction, endpoint restriction,
untrusted-input, finding-schema, changed-line, duplicate, and output-scan tests.

## Phase 5: Persistence, Audit, Operations, and Check Run

### T033 - Persist Review State Under Approved Retention

**Owner**: Persistence Engineer
**Depends on**: T004, T011, T030

- [ ] Persist allowed review state, finding summaries, non-sensitive evidence
  references, and configuration/model/rule versions.
- [ ] Implement retention expiry and repository-removal deletion.

**Acceptance Criteria**:

- Only approved data classes are persisted.
- Retention expiry and repository removal delete covered records according to
  the approved policy.
- Tests verify deletion without retaining source or prompts in backups/logs.

### T034 - Record Minimal Audit Events

**Owner**: Persistence Engineer
**Depends on**: T011, T033

- [ ] Record review identity, repository, PR/revision, event, versions, outcome,
  timing, and Check Run identifier.
- [ ] Exclude individual human identity and finding text from application audit
  records.

**Acceptance Criteria**:

- Required metadata is queryable by authorized operators.
- Audit data has access controls and follows the approved retention period.
- Tests inspect records and logs for source, prompts, secrets, PII, and finding
  text leakage.

### T035 - Implement Review State Machine

**Owner**: Worker Engineer
**Depends on**: T012, T016, T024, T028

- [ ] Implement `completed`, `partial`, `skipped`, `inconclusive`, and
  `failed` terminal states.
- [ ] Distinguish completed-with-no-findings from non-completed outcomes.

**Acceptance Criteria**:

- Every terminal transition has a defined cause and test.
- A clean completed review is distinguishable from skipped, partial,
  inconclusive, and failed reviews in stored state.
- Invalid state transitions are rejected or recorded as errors.

### T036 - Publish Non-Blocking Check Run Summary

**Owner**: GitHub Integration Engineer
**Depends on**: T018, T030, T035

- [ ] Publish one Check Run summary for each eligible review identity.
- [ ] Map terminal states to the specified success or neutral non-required
  conclusion and include a clear state summary.

**Acceptance Criteria**:

- Completed reviews, with or without findings, use a successful non-required
  Check Run.
- Partial, skipped, inconclusive, and failed reviews use a neutral non-required
  Check Run with explicit state.
- The Check Run is not required by branch protection and never blocks merges.

### T037 - Make Check Run Updates Idempotent

**Owner**: GitHub Integration Engineer
**Depends on**: T016, T036

- [ ] Update/reuse the Check Run for repeated delivery of the same review
  identity.
- [ ] Keep inline comment writes unavailable to the prototype installation.

**Acceptance Criteria**:

- Repeated webhook delivery does not create duplicate Check Runs.
- The GitHub App has no comment-write permission.
- A stale-head guard prevents an old review from overwriting a newer result.

### T038 - Add Basic Health and Operational Metrics

**Owner**: Platform Engineering
**Depends on**: T017, T035, T036

- [ ] Expose health status and counters for queue delay, duration, terminal
  state, retries, and provider failures.
- [ ] Assign operational triage to Platform Engineering.

**Acceptance Criteria**:

- Health endpoint distinguishes ready from unavailable dependencies.
- Metrics can be checked in automated tests or local operational output.
- Metrics and logs contain no source, prompt, secret, finding text, or human
  identity.

**Milestone M5 Gate**: T033-T038 pass retention, audit, state mapping,
Check Run idempotency, and non-blocking behavior tests.

## Phase 6: End-to-End Evaluation and Shadow Pilot

### T039 - Build End-to-End Test Harness

**Owner**: Test Engineer
**Depends on**: T014, T017, T020, T028, T033, T036

- [ ] Build signed webhook fixtures and a controlled model stub.
- [ ] Run the API, worker, and PostgreSQL integration path in tests.

**Acceptance Criteria**:

- A test event can be authenticated, processed, persisted, and reflected in a
  Check Run adapter result.
- Unauthorized, malformed, stale, and provider-failure cases are reproducible.
- The harness does not call a public model endpoint or use production secrets.

### T040 - Execute Pilot Acceptance and Security Tests

**Owner**: Test/Security Engineers
**Depends on**: T039

- [ ] Run all specification acceptance tests and cross-phase security/privacy
  tests in CI or a controlled integration environment.
- [ ] Record failures, evidence, and owners for remediation.

**Acceptance Criteria**:

- FR-001 through FR-026 and SC-001 through SC-010 applicable to prototype
  scope have test evidence.
- Seeded secrets are absent from model payloads, persistent records, logs, and
  Check Run summaries.
- No unresolved critical security/privacy failure remains before pilot use.

### T041 - Label and Report Evaluation Corpus

**Owner**: Evaluation Lead
**Depends on**: T006, T040

- [ ] Obtain independent reviewer labels and adjudicate disagreements.
- [ ] Produce metrics by category/severity with sample counts and denominators.

**Acceptance Criteria**:

- Tuning and evaluation pull requests are separate.
- Precision/actionability, false-positive share, valid-line anchoring,
  feedback, latency, completion state, and reliability are reported.
- No numeric quality gate or comment activation is inferred from these results.

### T042 - Start Single-Repository Shadow Pilot

**Owner**: Platform Engineering Lead
**Depends on**: T003, T004, T005, T007, T040, T041

- [ ] Enable the GitHub App only for the selected repository.
- [ ] Verify Check Run audience and non-required status before processing
  repository content.

**Acceptance Criteria**:

- Security and Privacy approvals, model approval, retention configuration, and
  repository policy are recorded before activation.
- No inline comments are posted and no merge/CI gate is introduced.
- Pilot volume remains within the approved in-process capacity boundary.

### T043 - Review Pilot Results and Close Milestone

**Owner**: Platform Engineering Lead
**Depends on**: T042

- [ ] Review quality, privacy/security events, failure states, and capacity
  observations with Platform, Security, Privacy, and the repository maintainer.
- [ ] Record follow-up decisions separately from this prototype's scope.

**Acceptance Criteria**:

- M6 exit criteria in `spec/plan.md` are met or each unmet criterion has an
  owner and explicit blocked status.
- Comment publication, commerce-specific rules, additional repositories, and
  scale-out remain disabled unless separately specified and approved.
- Pilot findings are not used as individual performance measures.

## Explicitly Excluded Tasks

- Inline PR comment publication or automatic comment-mode activation.
- Commerce-specific order, refund, cancellation, payment, or event checks.
- User-facing React screens, dashboards, artifacts, or configuration editors.
- Transitive dependency-graph expansion, mixed-language review, and
  unsupported-file analysis.
- Durable queue replay, horizontal scale-out, numeric SLO commitments, and
  advanced database backup/restore guarantees.

## Completion Definition

- Every in-scope task is complete or explicitly blocked with an owner and
  recorded reason.
- Milestones M0-M6 meet their exit criteria.
- No out-of-scope task has been introduced without an approved specification
  change.
- All specification and constitution constraints continue to pass the required
  tests and security/privacy review.
