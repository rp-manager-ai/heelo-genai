# Implementation Plan: Project- and Domain-Specific PR Reviewer

**Feature Branch**: `TBD`
**Created**: 2026-10-02
**Status**: Draft
**Specification**: `spec/specification.md` v1.0.0
**Constitution**: `spec/constitution.md` v1.0.0
**Owner**: Platform Engineering Lead (individual TBD)

## Summary

Deliver a single-repository, shadow-only prototype for GitHub Enterprise Cloud.
The service validates signed pull-request events, collects bounded revision-
pinned context, redacts sensitive values, calls only the approved private model,
validates evidence-backed findings, persists permitted metadata in PostgreSQL,
and publishes a non-blocking GitHub Check Run summary.

The prototype does not post inline comments, evaluate commerce-specific rules,
provide a user-facing React application, or promise horizontal scaling or an
availability/latency SLO. Those boundaries follow the confirmed decisions in
the specification.

## Repository Starting Point

- `apps/api/src/app.js`, `apps/worker/src/index.js`, and the React entrypoints
  are tracked placeholders and currently empty.
- `packages/config`, `packages/github`, `packages/model`,
  `packages/review-core`, and `packages/shared` have placeholder entrypoints.
- The root package currently has Express and Node's test runner, but no declared
  workspace setup, PostgreSQL client, React/Vite dependencies, or Docker
  service configuration.
- `server.js` and `test/server.test.js` are the existing runnable Express/test
  surface. Preserve or deliberately migrate their behavior while wiring the
  new app modules.

## Technical Approach

- Keep HTTP intake in `apps/api`; acknowledge verified events quickly and pass
  accepted work to `apps/worker` through the confirmed best-effort in-process
  queue.
- Put GitHub authentication and Check Run operations in `packages/github`;
  keep model transport in `packages/model` and orchestration, redaction, and
  validation in `packages/review-core`.
- Put YAML parsing, JSON Schema validation, organization defaults, and policy
  precedence in `packages/config`. Keep shared entities and versioned payload
  schemas in `packages/shared`.
- Use PostgreSQL 15 for review state, finding summaries, non-sensitive evidence
  references, configuration/model/rule versions, Check Run identifiers, and
  allowed audit metadata. Do not persist source, raw diffs, prompts, or full
  model traces.
- Use Docker for local PostgreSQL. Add only the runtime dependencies needed by
  these modules and document supported versions and configuration.
- Do not build operator screens or other user-facing pages. If frontend
  scaffolding is retained, it must follow the constitution's React 18/Vite
  constraint but remains outside the prototype's functional acceptance scope.

## Phases and Milestones

### Phase 0: Readiness and Scope Freeze

**Milestone M0: Pilot readiness approved**

1. Select the single pilot repository and confirm that its changes fit the
   TypeScript/Node.js and Terraform/Kubernetes/Helm changed-file scope.
2. Name the Platform Engineering Lead, Security Owner, Privacy Owner, and
   repository maintainer who will approve and operate the pilot.
3. Approve the private model provider, endpoint, deployment region, and data-
   processing terms before any repository content is sent for inference.
4. Approve retention and deletion periods for finding summaries and audit
   metadata. Define residency and repository-removal behavior.
5. Provide valid organization defaults and the selected repository's YAML
   policy. Confirm the configuration owner and the policy version to use.
6. Agree on a labeled historical/synthetic evaluation set and reviewer-label
   process. Identify the shadow Check Run audience.
7. Resolve the capacity boundary: the specification requires an in-process,
   best-effort queue while NFR-005 describes queue-based horizontal scaling.
   For this prototype, confirm acceptable pilot volume and concurrency; do not
   implement scale-out or durable replay under this plan.

**Exit criteria**: Required owners and approvals are named; model, privacy,
retention, configuration, pilot repository, and evaluation prerequisites are
recorded. No source-processing implementation proceeds without M0 approval.

### Phase 1: Runtime, Contracts, and Local Data Foundation

**Milestone M1: Local services and contracts are testable**

1. Define root scripts and package boundaries for the existing `apps/*` and
   `packages/*` structure. Keep JavaScript modules consistent with the current
   ESM setup unless the team explicitly approves a language/tooling change.
2. Establish a supported Node.js runtime and reproducible install/test
   commands. Keep the existing root health/chat tests passing or migrate them
   with an explicit compatibility decision.
3. Add a Docker-based PostgreSQL 15 development service, connection
   configuration, migration runner, and readiness/health check.
4. Define database migrations for review jobs, findings, Check Run state,
   configuration versions, and approved audit metadata. Add uniqueness and
   transaction boundaries for idempotency.
5. Define shared JSON Schemas for repository configuration, candidate/final
   findings, worker messages, and review outcomes. Version all persisted and
   queued contracts.
6. Add CI checks for tests, schema validation, migrations, and the repository
   markdown validator. Do not make the reviewer Check Run a required status
   check.

**Exit criteria**: A clean checkout can install dependencies, start PostgreSQL
through Docker, apply migrations, validate schemas, and run automated tests.

### Phase 2: Authenticated GitHub Intake and Revision Safety

**Milestone M2: Only authorized, revision-pinned events enter processing**

1. Implement the Express webhook route with raw-body signature verification,
   bounded request handling, and installation/repository authorization.
2. Accept non-draft `opened`, `reopened`, `synchronize`, and
   `ready_for_review` events. Ignore draft events until ready; cancel pending
   work on close/merge where possible.
3. Persist event/review identity and deduplication state using delivery ID,
   repository, pull-request number, and head SHA. Reject unauthorized events
   before reading repository content or enqueueing work.
4. Enqueue accepted work in the best-effort in-process queue. Bound active work
   and return a non-blocking acknowledgement; document that a process restart
   can lose pending jobs.
5. Before publishing any result, re-read or otherwise verify the current PR
   head and suppress output when it no longer matches the job SHA.
6. Implement bounded retry handling for transient GitHub failures, initially
   with at most one retry and bounded backoff. Do not retry permanent auth or
   validation failures.

**Exit criteria**: Tests prove valid events enqueue once, bad signatures and
unauthorized installations cannot access source, repeated delivery does not
duplicate work, and stale-head work cannot publish.

### Phase 3: Repository Policy and Bounded Context

**Milestone M3: A review receives only approved, bounded context**

1. Load `.pr-reviewer.yml` at the reviewed revision and validate it against
   JSON Schema version 1. Enforce organization policy over repository settings.
2. Apply valid organization defaults when repository configuration is missing
   or invalid. If valid defaults are unavailable, skip the review and record
   `configuration_unavailable` without blocking CI.
3. Retrieve the PR diff and supported changed files at the pinned head SHA.
   Include only explicitly referenced same-repository policy, ADR, and API
   contract documents; do not traverse dependency graphs.
4. Apply excluded-path rules and explicit bounds for file size, file count,
   diff size, and model context. Set initial numeric limits during M0 and store
   the effective limits with the review metadata.
5. Classify changed components and enabled general-risk categories. Skip
   commerce-specific order/refund/cancellation/event rules in this prototype.
6. Produce a context manifest recording included/excluded references and
   coverage state without persisting source content.

**Exit criteria**: Valid, invalid, and missing configuration cases are tested;
organization security controls cannot be overridden; context is revision-
pinned, bounded, and excludes configured paths; partial/skipped states explain
coverage limitations.

### Phase 4: Redaction, Model Analysis, and Finding Validation

**Milestone M4: Candidate findings are private, structured, and evidence-bound**

1. Implement baseline secret and configured-sensitive-data detection before
   any model request. Fail closed if redaction fails; do not send unredacted
   content to any model endpoint.
2. Add an outbound model adapter restricted to the approved private endpoint.
   Use bounded request size and timeout, structured input/output, and no model
   tool execution.
3. Treat code, comments, documentation, generated artifacts, and model output
   as untrusted. Add prompt-injection and malformed-output tests.
4. Validate candidate output against the versioned finding schema, category and
   severity enums, confidence range, concrete impact, rationale, and evidence.
5. Map each publishable finding to one or more relevant changed lines. Withhold
   findings supported only by unchanged context, unsupported claims, malformed
   evidence, or duplicate stable keys.
6. Scan output for secrets/sensitive values before persistence or Check Run
   publication. Persist only approved finding summaries and non-sensitive
   evidence references.

**Exit criteria**: Seeded-secret tests show no leakage in outbound model
payloads, persisted records, logs, or Check Run summaries. Tests cover
schema/evidence failures, changed-line mapping, prompt injection, duplicate
findings, unavailable model, and redaction failure.

### Phase 5: Persistence, Audit, Operations, and Check Run

**Milestone M5: Reviews are explainable and visible without merge impact**

1. Persist review states, finding summaries, evidence references, version
metadata, and Check Run identifiers under the approved retention policy.
2. Record the required audit fields: review ID, repository, PR and revision,
triggering event, configuration/model/rule versions, outcome, timing, and
Check Run identifier. Exclude source, prompts, secrets, finding text, and
individual human identities from application logs/audit data as specified.
3. Implement terminal states `completed`, `partial`, `skipped`,
`inconclusive`, and `failed`; distinguish completed-with-no-findings from
non-completed outcomes in persistence and user-visible output.
4. Publish one GitHub Check Run summary. Use a successful non-required result
for completed reviews, including no findings; use a neutral non-required result
with explicit state for partial, skipped, inconclusive, or failed reviews.
5. Make Check Run creation/update idempotent for the same review identity.
Do not request comment-write permission or publish inline comments.
6. Add basic health status and counters for queue delay, duration, review
state, retries, and provider failures. Assign operational triage to Platform
Engineering; a dashboard and response-time commitment remain out of scope.
7. Verify retention/deletion jobs and repository-removal handling before
loading pilot content.

**Exit criteria**: All terminal states map to the specified Check Run behavior;
Check Runs do not gate branch protection; logs and persistent data pass privacy
checks; retention and deletion behavior is approved and tested.

### Phase 6: End-to-End Evaluation and Shadow Pilot

**Milestone M6: Single-repository shadow pilot is accepted**

1. Build the integration test harness using signed webhook fixtures, a GitHub
   test installation/repository, a controlled model stub, and PostgreSQL 15.
2. Run unit, integration, security, privacy, migration, and failure-path tests
   against the acceptance criteria in `spec/specification.md`.
3. Evaluate the agreed corpus of historical and synthetic PRs containing both
   known-risk and clean changes. Keep tuning examples separate from evaluation
   examples by PR.
4. Have two reviewers label each evaluated case independently and send
   disagreements to the designated senior reviewer for adjudication.
5. Report precision/actionability, false-positive share, valid-line anchoring,
   acceptance/dismissal feedback, latency, completion states, and provider
   reliability by category and severity. Always include sample counts and
   denominators; do not claim numeric promotion thresholds.
6. Run the selected repository in shadow mode. Confirm no inline comments,
   merge blocking, required status checks, or unauthorized source/model access.
7. Review results and operational issues with Platform, Security, Privacy, and
   the repository maintainer. Keep comment publication and repository
   expansion disabled.

**Exit criteria**: All Initial Pilot Acceptance Criteria pass; Security and
Privacy owners approve configuration and data handling; retention is active;
the selected repository and evaluation results are documented. Any failure
that risks data exposure or merge blocking stops the pilot.

## Milestone Summary

| Milestone | Outcome | Required gate |
| --- | --- | --- |
| M0 | Readiness | Owners, model, retention, repository, policy |
| M1 | Local foundation | Install, PostgreSQL, migrations, tests |
| M2 | Secure intake | Auth, deduplication, stale-head tests |
| M3 | Context ready | Schema, bounds, exclusions, coverage tests |
| M4 | Validated analysis | Redaction, evidence, output-safety tests |
| M5 | Shadow publication | Retention, states, Check Run tests |
| M6 | Pilot accepted | Spec criteria and owner approvals |

## Cross-Phase Test Strategy

- **Unit tests**: configuration precedence, schema validation, redaction,
  changed-line mapping, stable finding keys, state transitions, and check
  conclusion mapping.
- **Integration tests**: webhook signature/auth, GitHub adapter behavior,
  worker processing, PostgreSQL transactions/migrations, and Check Run
  idempotency.
- **Security/privacy tests**: invalid signatures, unauthorized repositories,
  seeded secrets, prompt injection, model endpoint allowlisting, output
  scanning, log inspection, and retention/deletion.
- **Resilience tests**: GitHub/model timeouts, one-retry limit, queue saturation,
  process restart loss behavior, unavailable configuration, and stale PR heads.
- **Evaluation tests**: labeled clean and risky PRs, unsupported evidence,
  category/severity segmentation, reviewer agreement, and denominator reporting.

## Risks and Plan Constraints

- **Queue scale conflict**: NFR-005 describes queue-based horizontal scaling;
  confirmed prototype scope specifies a best-effort in-process queue and marks
  durable replay/scale-out out of scope. M0 must record acceptable pilot
  traffic/concurrency and the resulting single-process limitation. If the pilot
  needs horizontal scaling, this plan and the specification require revision.
- **Approval dependencies**: No source processing starts before approved model,
  data-processing terms, retention, and named Security/Privacy owners.
- **Configuration dependency**: No review starts without valid organization
  defaults; missing defaults result in a skipped, non-blocking check.
- **Frontend boundary**: React 18/Vite user-facing screens are out of scope;
  do not let frontend scaffolding expand into an operator console.
- **Future features**: Inline comments, commerce-specific rules, dashboards,
  multi-repository onboarding, durable queues, and SLO commitments require a
  separate reviewed specification and owner approval.

## Deferred Decisions

- Exact pilot repository, model endpoint, deployment region, and approved
  data-processing terms.
- Initial numeric bounds for file count, diff size, context size, timeouts, and
  permitted in-process concurrency.
- Approved retention/deletion period and data residency.
- Evaluation corpus size and pilot duration; numeric quality-promotion gates
  are not part of this prototype.
- Resolution of the NFR-005 horizontal-scaling conflict if measured pilot needs
  exceed the accepted single-process capacity.
