# Task and Plan Analysis

**Reviewed**: `spec/constitution.md`, `spec/specification.md`, `spec/plan.md`,
and `spec/tasks.md`
**Review date**: 2026-10-02
**Task count**: 43
**Overall readiness**: T001 is complete by project-owner confirmation.
Several spec/task mismatches should still be resolved before source processing
or pilot activation.

## Executive Findings

1. **T001 is complete.** The project owner selected
   `raviepam/jira-confluence-automation`, confirmed GitHub Enterprise Cloud
   hosting and supported file-scope fit, and identified RP as repository
   maintainer and installation owner. The initial mismatch between the
   workspace name and remote was resolved explicitly rather than inferred.
2. **Scale requirements conflict.** NFR-005 asks for queue-based horizontal
   scaling while the confirmed prototype uses a best-effort in-process queue
   and excludes scale-out. T007 calls for a capacity decision, but the
   specification still retains both statements.
3. **Scope language is stale in user stories.** User Story 2 promises
   dependency-aware full-change review although FR-005/FR-005a and C-007 limit
   context to changed files and explicitly referenced documents. User Story 5
   is a future comment feature, but its acceptance scenarios look executable
   unless readers honor its prototype-status note.
4. **Some task dependencies do not match implementation order.** T019 tests
   model retries before T028 creates the model adapter. T008's workspace setup
   depends on the capacity decision T007 even though the work can be done
   without source processing. T030 finding-schema validation could proceed
   from T012 without waiting for T028.
5. **Several required controls lack implementation tasks.** NFR-001 requires
   encryption in transit and at rest; no task has acceptance criteria for
   either. The task list also does not assign a concrete general-risk
   analysis/rubric artifact, despite FR-007 and FR-026 requiring risk
   classification and useful findings.

## Recommended Execution Order

The order below respects the M0 source-processing gate while moving low-risk,
high-leverage decisions and foundation work early.

1. T001 is complete. Complete T002 role assignment; only the repository
   maintainer and installation owner are currently identified as RP.
2. After T002, complete T003 and T004 in parallel. Complete T005, T006, and
   T007 using the selected repository. This closes the M0 security, privacy,
   repository, policy, evaluation, and capacity gates.
3. After M0, start T008. Decouple it from T007 if the team agrees that
   workspace-only work is not source processing. Then run T009, T010, and T012
   in parallel. T012 is a low-risk, high-gain contract task that enables API,
   worker, config, and model work to proceed consistently.
4. Complete T011 after T010 and T004; complete T013 after T009, T011, and T012.
   Keep dependency/security/license review in this foundation gate.
5. Implement T014, T015, T016, T017, and T018 in dependency order. Reconcile
   the deduplication identity before T016. Limit T019 to GitHub retry behavior
   here; add model retry tests with T028/T029.
6. Implement T020/T021, then T022/T023/T024. T025's general-risk policy must
   be specified before analysis is considered complete; T026 follows the
   context-manifest contract.
7. After T027, implement T028/T029. Start T030 as soon as T012 is complete,
   parallel to the model adapter, instead of making schema validation wait for
   T028. Complete T031/T032 after their inputs are ready.
8. Complete T033/T034/T035, then T036/T037/T038. Do not enable pilot data until
   retention, encryption, audit access, and Check Run behavior pass review.
9. Finish T039/T040, then T041. Only then execute T042 and T043, with every M0
   approval still valid and the repository's pilot capacity respected.

**Low-risk, high-gain candidates**: T002 (unblocks approvals), T009 (protects
existing behavior), T012 (unblocks parallel contract work), and T013 (catches
regressions). T006 also has high leverage, but depends on the unresolved T001.
Security, privacy, and repository approvals remain gates; low implementation
cost is not a reason to bypass them.

## Cross-Document Gaps and Contradictions

### A-001 - Pilot Repository Selection Is Not Verifiable

**Status**: Resolved by project-owner confirmation
**References**: T001, M0, specification FR-001/FR-008 and Risks/Open Decisions.

The selected repository is `raviepam/jira-confluence-automation`. The project
owner confirmed GHE Cloud hosting and supported file-scope fit, and identified
RP as repository maintainer and GitHub App installation owner. Verify that the
actual GitHub App installation matches this record before T014 reads source.

### A-002 - Horizontal Scaling Versus In-Process Queue

**Severity**: High
**References**: Constitution delivery-safety principle; specification NFR-003,
NFR-005, C-009; plan M0 and Risks; tasks T007/T017.

NFR-005 still requires queue-based horizontal scaling, while the confirmed
prototype explicitly excludes scale-out and uses an in-process queue that can
lose pending work on restart. M0 can record acceptable volume, but does not
amend the requirement. Either revise NFR-005 for the prototype or explicitly
record an accepted deviation and numeric capacity boundary before pilot use.

### A-003 - User Story 2 Exceeds Context Scope

**Severity**: Medium
**References**: specification User Story 2, FR-005/FR-005a, C-007; plan Phase 3;
tasks T022/T023.

User Story 2 still says the reviewer considers “relevant dependencies” and the
“full change.” The implemented scope excludes dependency-graph traversal and
limits context to changed files plus explicitly referenced documents. Rewrite
the story and independent test to match that bounded behavior.

### A-004 - Delivery-ID Deduplication May Not Deduplicate a Revision

**Severity**: High
**References**: specification SC-002 and C-008; tasks T016/T037.

C-008 includes webhook delivery ID in the deduplication identity. If equivalent
events for the same repository, PR, and head SHA have different delivery IDs,
they can still start duplicate reviews. Define separately an event-delivery
key and a canonical review-revision key, including when a deliberate rerun is
allowed. Add concurrent duplicate tests for both keys.

### A-005 - Model Retry Task Precedes the Model Adapter

**Severity**: Medium
**References**: plan Phases 2/4; tasks T019, T028, and T029.

T019 claims GitHub/model retry behavior but depends only on intake and queue
work; the model adapter is T028. Split T019 to implement GitHub retries and
define the shared policy, then validate model retries alongside T028/T029.

### A-006 - General-Risk Analysis Has No Concrete Policy Artifact

**Severity**: High
**References**: specification FR-007, FR-013, FR-026 and categories; plan Phase
4; tasks T025/T028/T030.

T025 classifies categories and T028 transports model requests, but no task owns
the initial general-risk rubric, prompt template, category-specific evidence
rules, or expected clean/risky examples. Without these artifacts, the system
can call a model and validate structure without satisfying its review purpose.
Add a versioned review-policy/prompt artifact and tests per enabled category.

### A-007 - Encryption Requirements Have No Acceptance-Tested Task

**Severity**: High
**References**: specification NFR-001; plan Phases 1/5; tasks T010/T011/T033.

NFR-001 requires encryption in transit and at rest, but Docker PostgreSQL,
application connections, managed storage, and backups have no encryption
acceptance criteria. Assign ownership and tests for TLS/network boundaries and
storage encryption before storing pilot metadata.

### A-008 - Audit Actor Identity Is Ambiguous

**Severity**: Medium
**References**: specification Key Entity “Audit Event,” C-013, FR-024/FR-025;
tasks T034.

The Audit Event entity mentions “actor/source identity,” while C-013 excludes
individual human identity from application records. Define whether actor means
GitHub App installation/service identity only, then align the entity and audit
schema. Keep user identity in GitHub/organization audit systems if needed.

### A-009 - Repository Opt-Out and Feedback Are Not Prototype Tasks

**Severity**: Medium
**References**: specification FR-018, User Story 4, Scope; tasks T006/T041 and
Explicitly Excluded Tasks.

FR-018 requires repository opt-out and feedback controls before expanding the
comment pilot. The prototype is single-repository and has reviewer labels, but
there is no opt-out mechanism or feedback control. Mark FR-018 explicitly as
future/out of scope or add a task before any expansion.

### A-010 - PR Close Cancellation Is Weakened in Task Acceptance

**Severity**: Medium
**References**: specification C-008; plan Phase 2; tasks T015/T018.

C-008 says a close event cancels pending work. T015 acceptance criteria weaken
that to cancellation “where possible,” without requiring suppression of
publication if work cannot be stopped. Specify a closed/merged publication
guard and test it independently from stale-head suppression.

### A-011 - Shadow Finding Confidence/Visibility Is Unspecified

**Severity**: Medium
**References**: specification User Story 3, FR-014/FR-016/FR-017, C-004/C-014;
tasks T030/T036.

The story calls for low-confidence filtering, but the prototype has no numeric
quality threshold and still places validated findings in a Check Run. Define
whether low-confidence candidates are included in shadow output, suppressed,
or retained only for private evaluation. Keep numeric comment-promotion gates
out of scope as confirmed.

### A-012 - Encryption, Policy, and Test Artifacts Lack Stable Paths

**Severity**: Medium
**References**: plan repository approach; tasks T010/T012/T020/T027/T040.

Tasks require Docker configuration, migrations, OpenAPI, JSON Schemas, org
defaults, redaction policy, and CI checks without naming their repository paths
or ownership boundaries. Agree on artifact locations during T008/T012/T020 so
the plan is reviewable and generated files are not scattered.

### A-013 - Frontend Constitution Versus Prototype Scope Needs One Boundary

**Severity**: Low
**References**: constitution Technology Constraints; specification C-015;
plan Technical Approach; tasks T008/T009.

The constitution mandates React 18/Vite for a frontend; C-015 excludes
user-facing React screens. The plan says scaffolding is conditional, and no
task adds frontend dependencies. Confirm that no frontend is required in this
prototype, or add only the minimum React/Vite shell needed to satisfy the
constitution without adding UI scope.

### A-014 - Task Traceability Is Not Explicit

**Severity**: Low
**References**: specification FR/SC IDs; tasks T040.

T040 asks for evidence against all applicable FRs and SCs, but the task list
does not assign a requirements-to-tests traceability artifact. Add a compact
matrix mapping each in-scope FR/SC to tests and explicitly label out-of-scope
requirements.

## Task-by-Task Assessment

Complexity estimates reflect implementation effort and integration breadth.
Risk estimates reflect the impact of a wrong decision or defect.

### Phase 0: Readiness and Scope Freeze

#### T001 - Select Pilot Repository

**Complexity**: Low. **Risk**: High. **Dependencies**: None.
**Risk**: Wrong repository selection could invalidate pilot authorization.
Owner confirmation resolves selection; verify installation access at T014.

#### T002 - Assign Pilot Approval Roles

**Complexity**: Low. **Risk**: Medium. **Dependencies**: None.
**Risk**: Unowned approvals delay the pilot or leave security/privacy decisions
without accountable reviewers. High-gain first action.

#### T003 - Approve Private Model Boundary

**Complexity**: Medium. **Risk**: High. **Dependencies**: T002.
**Risk**: Incorrect endpoint, region, or terms can expose source code or
violate enterprise policy. Requires security/provider coordination.

#### T004 - Approve Data Retention and Residency

**Complexity**: Medium. **Risk**: High. **Dependencies**: T002.
**Risk**: Undefined retention or residency can create privacy/legal exposure
and invalidate persistence design. Pilot gate, not an implementation guess.

#### T005 - Provide Organization Policy Defaults

**Complexity**: Medium. **Risk**: High. **Dependencies**: T001, T002.
**Risk**: Unsafe or missing defaults could analyze repositories under weaker
controls or produce silent skips. Requires versioned policy and owner.

#### T006 - Approve Evaluation Set and Shadow Audience

**Complexity**: Medium. **Risk**: Medium. **Dependencies**: T001, T002.
**Risk**: Biased examples, leaked source, or an overly broad Check Run audience
can distort quality results or expose findings.

#### T007 - Set Single-Process Capacity Boundary

**Complexity**: Medium. **Risk**: High. **Dependencies**: T001.
**Risk**: An unbounded pilot can overload a lossy in-process queue; this task
also exposes the NFR-005 scale conflict.

### Phase 1: Runtime, Contracts, and Local Data Foundation

#### T008 - Define Workspace and Module Boundaries

**Complexity**: Medium. **Risk**: Medium. **Dependencies**: T007.
**Risk**: Workspace rewiring can break the existing Express server/tests or
create circular ownership. Dependency on T007 appears unnecessary for
scaffolding-only work.

#### T009 - Pin Runtime and Preserve Existing Tests

**Complexity**: Low. **Risk**: Medium. **Dependencies**: T008.
**Risk**: Runtime drift or accidental removal of existing health/chat behavior
can destabilize development. Early regression protection has high gain.

#### T010 - Add Local PostgreSQL 15 Service

**Complexity**: Medium. **Risk**: Medium. **Dependencies**: T008.
**Risk**: Credential leakage, insecure local exposure, or environment drift can
make later integration tests unreliable.

#### T011 - Add Migrations and Persistence Constraints

**Complexity**: High. **Risk**: High. **Dependencies**: T010, T004.
**Risk**: Incorrect uniqueness, retention, or data modeling can cause duplicate
reviews, data loss, or retention violations.

#### T012 - Define Versioned Service Contracts

**Complexity**: Medium. **Risk**: Medium. **Dependencies**: T008.
**Risk**: Inconsistent schemas across API/worker/model create runtime failures.
This is a high-gain early artifact and can unblock parallel implementation.

#### T013 - Establish CI Quality Gates

**Complexity**: Medium. **Risk**: Medium. **Dependencies**: T009, T011, T012.
**Risk**: A misconfigured required check could block merges; CI omissions let
schema or migration regressions through.

### Phase 2: Authenticated GitHub Intake and Revision Safety

#### T014 - Implement Signed Webhook Endpoint

**Complexity**: High. **Risk**: High. **Dependencies**: T002, T008, T012.
**Risk**: Signature/body parsing or installation authorization mistakes could
allow unauthorized source access. Explicitly depend on T001/M0 pilot selection.

#### T015 - Implement Event and Draft Lifecycle

**Complexity**: Medium. **Risk**: High. **Dependencies**: T014.
**Risk**: Incorrect event/action handling can miss reviews or process drafts;
closed/merged work must never publish after cancellation fails.

#### T016 - Persist Event Identity and Deduplicate

**Complexity**: High. **Risk**: High. **Dependencies**: T011, T014.
**Risk**: A weak idempotency key or non-atomic enqueue can create duplicate
jobs. Resolve delivery identity versus canonical revision identity first.

#### T017 - Add Bounded In-Process Queue

**Complexity**: Medium. **Risk**: High. **Dependencies**: T007, T008, T016.
**Risk**: Process restart loses pending work; saturation or poor limits can
consume memory and delay webhook acknowledgement.

#### T018 - Guard Against Stale Head Publication

**Complexity**: High. **Risk**: High. **Dependencies**: T015, T016.
**Risk**: Race conditions can publish findings for obsolete code. Also needs a
closed/merged guard, not only a head-SHA comparison.

#### T019 - Implement Bounded Retry Policy

**Complexity**: Medium. **Risk**: Medium. **Dependencies**: T014, T017.
**Risk**: Unbounded or misclassified retries amplify provider outages. Model
retry implementation is premature until T028/T029.

### Phase 3: Repository Policy and Bounded Context

#### T020 - Validate Repository YAML

**Complexity**: Medium. **Risk**: High. **Dependencies**: T005, T012.
**Risk**: Parser/schema or precedence errors can bypass mandatory policy.
Schema path, versioning, and test fixtures must be stable.

#### T021 - Implement Safe Defaults and Skip Behavior

**Complexity**: Medium. **Risk**: High. **Dependencies**: T020.
**Risk**: A permissive fallback can silently weaken controls; a wrong skip state
can hide failures. Verify invalid defaults never reach inference.

#### T022 - Collect Revision-Pinned Changed Files

**Complexity**: High. **Risk**: High. **Dependencies**: T014, T018, T020.
**Risk**: Wrong-revision or unauthorized file retrieval can leak data or ground
findings in stale code; GitHub API limits add reliability risk.

#### T023 - Load Explicit Context and Apply Exclusions

**Complexity**: Medium. **Risk**: High. **Dependencies**: T020, T022.
**Risk**: Misresolved references or exclusion bypass can expose prohibited
content. References must be revision-pinned and bounded.

#### T024 - Enforce Context Bounds and Coverage States

**Complexity**: Medium. **Risk**: High. **Dependencies**: T007, T022, T023.
**Risk**: Limits that are too high risk resource exhaustion; limits that are
too low cause misleading partial reviews.

#### T025 - Classify General Review Categories

**Complexity**: Medium. **Risk**: Medium. **Dependencies**: T020, T022.
**Risk**: Category misclassification can suppress relevant review. Classification
alone does not implement a general-risk analysis rubric; see A-006.

#### T026 - Produce Non-Sensitive Context Manifest

**Complexity**: Low. **Risk**: Medium. **Dependencies**: T023, T024.
**Risk**: A manifest may contain paths or metadata that policy classifies as
sensitive. It has diagnostic value at low effort.

### Phase 4: Redaction, Model Analysis, and Finding Validation

#### T027 - Redact Sensitive Values Before Inference

**Complexity**: High. **Risk**: High. **Dependencies**: T003, T023, T024.
**Risk**: False negatives expose secrets/PII; false positives erase essential
context. Failure must stop inference, not merely emit a warning.

#### T028 - Add Approved Private Model Adapter

**Complexity**: High. **Risk**: High. **Dependencies**: T003, T012, T027.
**Risk**: Endpoint misconfiguration, data-processing behavior, timeouts, or
provider-specific output drift can expose data or break analysis.

#### T029 - Test Untrusted Input and Model Failures

**Complexity**: Medium. **Risk**: High. **Dependencies**: T028.
**Risk**: Prompt injection or malformed outputs may bypass controls if fixtures
cover only obvious attacks. Use adversarial and regression examples.

#### T030 - Validate Finding Schema and Required Fields

**Complexity**: Medium. **Risk**: Medium. **Dependencies**: T012, T028.
**Risk**: Weak validation admits unsupported findings; overvalidation can drop
legitimate results. This can start after T012, parallel to T028.

#### T031 - Map Findings to Changed Lines and Deduplicate

**Complexity**: High. **Risk**: High. **Dependencies**: T022, T030.
**Risk**: Incorrect line mapping violates the Constitution and invalidates
inline-ready evidence; unstable keys cause repeated findings.

#### T032 - Scan Output Before Storage and Publication

**Complexity**: High. **Risk**: High. **Dependencies**: T027, T030.
**Risk**: Model output can repeat sensitive data even after input redaction.
Withholding is safer than inventing an undefined sanitizer policy.

### Phase 5: Persistence, Audit, Operations, and Check Run

#### T033 - Persist Review State Under Approved Retention

**Complexity**: High. **Risk**: High. **Dependencies**: T004, T011, T030.
**Risk**: Persistence can retain prohibited content or outlive approved
retention. Deletion must include dependent records and backups where applicable.

#### T034 - Record Minimal Audit Events

**Complexity**: Medium. **Risk**: High. **Dependencies**: T011, T033.
**Risk**: Missing events weaken auditability; excess fields expose identities,
source, or finding text. Align actor semantics with C-013.

#### T035 - Implement Review State Machine

**Complexity**: Medium. **Risk**: High. **Dependencies**: T012, T016, T024,
T028.
**Risk**: Invalid transitions can report a clean review after partial or failed
analysis. State causes and terminal behavior need exhaustive tests.

#### T036 - Publish Non-Blocking Check Run Summary

**Complexity**: High. **Risk**: High. **Dependencies**: T018, T030, T035.
**Risk**: Wrong conclusion or required branch protection can block delivery;
Check Run permissions and state mapping need integration tests.

#### T037 - Make Check Run Updates Idempotent

**Complexity**: Medium. **Risk**: High. **Dependencies**: T016, T036.
**Risk**: Duplicate Check Runs or stale updates confuse users. Ensure write
permissions exclude comments and key behavior matches T016.

#### T038 - Add Basic Health and Operational Metrics

**Complexity**: Medium. **Risk**: Medium. **Dependencies**: T017, T035, T036.
**Risk**: Misleading health can hide provider/queue failure; metric fields must
not leak repository-sensitive content.

### Phase 6: End-to-End Evaluation and Shadow Pilot

#### T039 - Build End-to-End Test Harness

**Complexity**: High. **Risk**: High. **Dependencies**: T014, T017, T020, T028,
T033, T036.
**Risk**: Test doubles can diverge from GitHub/model behavior; production
credentials or public model endpoints must never enter the harness.

#### T040 - Execute Pilot Acceptance and Security Tests

**Complexity**: High. **Risk**: High. **Dependencies**: T039.
**Risk**: “Applicable” FR/SC criteria may be omitted or interpreted
inconsistently. Requires a traceability matrix and recorded security review.

#### T041 - Label and Report Evaluation Corpus

**Complexity**: Medium. **Risk**: Medium. **Dependencies**: T006, T040.
**Risk**: Reviewer disagreement, leakage, or biased corpus can make metrics
misleading. Preserve separation and report denominators.

#### T042 - Start Single-Repository Shadow Pilot

**Complexity**: High. **Risk**: High. **Dependencies**: T003, T004, T005, T007,
T040, T041.
**Risk**: A missing approval or incorrect installation exposes repository data
or blocks delivery. Require M0 gate and explicit T001/T002 completion.

#### T043 - Review Pilot Results and Close Milestone

**Complexity**: Medium. **Risk**: Medium. **Dependencies**: T042.
**Risk**: Pressure to expand scope before evidence is adequate. Record unmet
criteria as blocked and keep out-of-scope features disabled.

## Missing Artifacts to Assign

The task list names deliverables but often not their paths or owners. Assign
locations during T008/T012/T020 so reviewers can find the source of truth.

- OpenAPI document for the webhook/API contract.
- Versioned JSON Schemas for config, findings, worker messages, and outcomes.
- Example `.pr-reviewer.yml` and organization-default policy fixture.
- Docker Compose/service definition, migration directory, and database
  connection/secret configuration.
- Versioned general-risk rubric and model prompt/template with category test
  fixtures.
- Redaction policy/test fixtures and output scan policy.
- Threat model/data-flow diagram and security review record.
- CI workflow and FR/SC-to-test traceability matrix.
- Evaluation corpus manifest, labeling instructions, and metric report format.
- Retention/deletion configuration and operational runbook for the pilot.

## Recommended Task-Graph Adjustments

- T001 is complete. Verify the selected repository and RP installation
   ownership against the actual GitHub App installation before T014 proceeds.
- Allow T008 workspace/bootstrap work to proceed independently of T007, provided
  it does not process source or start pilot workloads.
- Split T019: implement GitHub retry behavior in Phase 2; implement/test model
  retry behavior with T028/T029 in Phase 4.
- Allow T030 to start after T012 and run parallel to T028.
- Add a close/merge publication guard to T015/T018, not just best-effort job
  cancellation.
- Add a general-risk rubric/prompt artifact and tests before claiming T025
  completes analysis behavior.
- Add explicit encryption-in-transit/at-rest acceptance criteria and a
  requirements-to-tests traceability artifact before T040.
- Make T042 explicitly depend on completion of the entire M0 gate, including
  T001, T002, and T006, rather than relying only on indirect dependencies.
