# Project Constitution

**Version:** 1.0.0
**Ratified:** 2026-10-02
**Last Amended:** 2026-10-02

## Purpose

This constitution sets heelo-genai's non-negotiable engineering principles. It governs an
AI-assisted pull request reviewer that supports engineers without replacing human judgment.

## Principles

### 1. Human-Owned Review Decisions

- The reviewer MUST NOT approve, reject, or merge pull requests autonomously.
- The initial release MUST NOT change source code or block merges.
- Findings MUST be actionable; speculative or style-only comments MUST be suppressed.

### 2. Privacy and Security by Design

- Repository content MUST stay within approved enterprise infrastructure; public model endpoints
  MUST NOT receive it.
- Secrets and configured sensitive data MUST be redacted before model requests.
- Logs MUST exclude raw source, secrets, and unnecessary personal data.
- Webhooks and GitHub operations MUST use authenticated, least-privilege access.
- Credentials MUST stay outside source control and repository configuration.
- Repository content and model output MUST remain untrusted and cannot override policy or
  authorize tools.

### 3. Evidence Before Findings

- Every finding MUST cite a changed line, concrete impact, severity, confidence, and evidence.
- Findings MUST pass schema validation and align with relevant project guidance or contracts.
- Missing context or uncertain evidence MUST produce a partial or inconclusive result, not an
  invented finding.
- Reviews MUST be pinned to a pull-request head SHA and deduplicated across retries.

### 4. Fail Safely and Stay Non-Blocking

- GitHub, model, or database failures MUST NOT block normal delivery.
- Retries and resource use MUST be bounded; exhausted retries MUST be observable and non-blocking.
- Shadow mode MUST NOT publish inline comments.
- Comment mode MUST publish only findings that pass the agreed quality threshold.

### 5. Modular Contracts and Change Safety

- Frontend, API, worker, model, and persistence responsibilities MUST have explicit contracts.
- API, configuration, and database changes MUST be validated and versioned as compatibility needs.
- Database schema changes MUST use reviewed migrations that consider existing data and app versions.
- Provider-specific model and GitHub behavior SHOULD use replaceable adapters.

### 6. Test the Risk, Observe the System

- Changes MUST include risk-appropriate automated tests for failure, retry, authorization, and data
  boundaries.
- Tests MUST ensure known secrets do not reach model payloads or ordinary logs.
- Operational records MUST support audits and diagnosis without retaining unnecessary source data.
- Shadow-mode quality MUST measure actionability, false positives, line validity, and reliability
  before comment publication expands.

## Technology Constraints

- Frontend MUST use React 18 with Vite.
- Backend MUST use Node.js with Express.
- Database MUST use PostgreSQL 15, run through Docker where containers are used.
- New dependencies MUST be justified, supported, and reviewed for security and license suitability.
- Secrets and environment-specific configuration MUST NOT be committed to the repository.

## Engineering Workflow

- Changes MUST preserve these principles and update tests or documentation when behavior changes.
- Pull requests MUST describe relevant behavior, security/privacy effects, migrations, and
  operational impact.
- CI MUST validate changed code and contracts; the reviewer MUST remain advisory and non-blocking.
- Security, model endpoint, retention, and publication-policy changes require owner review.

## Governance

- Applicable legal, security, and organization policies remain controlling.
- Amendments MUST document rationale and impact, then update the version: major for breaking
  changes, minor for new principles, and patch for editorial corrections.
- Specifications, plans, and pull requests MUST resolve conflicts with this constitution before
  implementation.
