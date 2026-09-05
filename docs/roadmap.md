# Roadmap

Kestrel is being developed incrementally.

The roadmap favors useful, testable milestones over speculative deadlines. Features should expand
Kestrel's capabilities without weakening its authority, privacy, or least-privilege boundaries.

Completed work is recorded here so the roadmap also communicates what the current prototype actually
does.

## Current working prototype

The first deterministic task-planning pipeline is operational.

- [x] Establish repository hygiene, security documentation, and secret scanning.
- [x] Create a reproducible Docker Compose development stack.
- [x] Run an isolated Ollama service for future local inference.
- [x] Deploy Activepieces with a separate worker and sandboxed Code execution.
- [x] Connect to selected Nextcloud Tasks collections through a dedicated account.
- [x] Enforce read-only access to canonical Nextcloud task content.
- [x] Normalize relevant CalDAV/VTODO task metadata.
- [x] Define deterministic task date, priority, age, and stale-review semantics.
- [x] Implement the deterministic daily planner.
- [x] Distinguish Personal and Classes planning behavior.
- [x] Implement mandatory deadline windows and future-start suppression.
- [x] Implement scoring for priority, class work, deadline proximity, elapsed work window, and age.
- [x] Implement rotating stale-backlog review.
- [x] Project the daily plan into Google Tasks as `Kestrel Today`.
- [x] Preserve source task identity and selection reasoning in projection metadata.
- [x] Reconcile Google projections idempotently.
- [x] Reorder Google projections to match planner ranking.
- [x] Keep unmanaged Google tasks outside Kestrel reconciliation.
- [x] Externalize deployment-specific Nextcloud and timezone configuration.
- [x] Dynamically resolve the `Kestrel Today` Google list rather than hardcoding its ID.
- [x] Publish a sanitized, importable Activepieces flow template.
- [x] Document planner policy, setup requirements, architecture, and security boundaries.
- [x] Document AI-assisted development and attribution.

The canonical flow is:

```text
flows/build-kestrel-today.json
```

At this stage:

```text
Nextcloud Tasks
    = authoritative

Google "Kestrel Today"
    = derived projection
```

The current planner is deterministic and does not require an LLM at runtime.

## Reproducible onboarding

The immediate goal is to make the working prototype practical for another technically competent
person to deploy.

- [x] Create a portable Activepieces flow template.
- [x] Remove source deployment secrets and connection identifiers from the public template.
- [x] Document required Activepieces project variables.
- [x] Document Nextcloud least-privilege requirements.
- [x] Document Google Tasks configuration and first-run validation.
- [x] Create an LLM-guided setup prompt.
- [x] Update the README to describe the working prototype and recommended setup path.
- [x] Review documentation for consistency and remove stale prototype-era instructions.
- [ ] Perform a clean-install walkthrough using only the public repository and guided setup
  material.
- [ ] Record any compatibility limitations discovered during a clean installation.

The intended onboarding model is:

```text
Clone repository
      |
      v
Deploy services
      |
      v
Import tested flow
      |
      v
Configure local variables/connections
      |
      v
Validate manually
      |
      v
Enable scheduled execution
```

Users should not normally reconstruct the Activepieces flow by hand.

## Task capture

Kestrel currently projects tasks outward to Google but does not yet use Google as an inbound capture
surface.

The planned capture model is intentionally asymmetric.

### `Kestrel Inbox`

- [ ] Create a Google Tasks list named `Kestrel Inbox`.
- [ ] Treat it as a user/voice/mobile capture surface.
- [ ] Read newly captured items deterministically.
- [ ] Normalize captured task metadata.
- [ ] Detect already-imported items and avoid duplicate canonical tasks.
- [ ] Propose or create corresponding canonical Nextcloud tasks through an explicitly controlled
  path.
- [ ] Preserve provenance from Google capture to the resulting Nextcloud task.
- [ ] Define what happens to successfully imported Google capture items.

This should not become generic bidirectional synchronization.

Conceptually:

```text
Google "Kestrel Inbox"
        |
        v
Kestrel ingestion
        |
        v
Nextcloud Tasks
    canonical
```

while the existing daily projection remains:

```text
Nextcloud Tasks
        |
        v
Kestrel planner
        |
        v
Google "Kestrel Today"
```

## Calendar-aware planning

Calendar awareness remains an important part of the original Kestrel planning goal.

- [ ] Connect to approved calendar sources with read-only access.
- [ ] Normalize relevant calendar events.
- [ ] Estimate daily scheduled load.
- [ ] Estimate usable discretionary work capacity.
- [ ] Reduce discretionary planner output on heavily scheduled days.
- [ ] Preserve mandatory deadlines regardless of calendar load.
- [ ] Avoid interpreting calendar descriptions as trusted instructions.
- [ ] Document calendar-planning policy separately from hard task rules.

Calendar load should influence discretionary planning rather than hiding genuinely urgent work.

## Task semantics and planner improvements

The deterministic planner is useful but intentionally incomplete.

Potential additions include:

- [ ] Distinguish estimated effort from required lead time.
- [ ] Define optional effort metadata such as Quick, Medium, Deep, and MultiDay.
- [ ] Define lead-time metadata for work that must begin several days before its deadline.
- [ ] Represent blocked or waiting tasks.
- [ ] Account for task dependencies.
- [ ] Improve diversity across projects or task categories when appropriate.
- [ ] Add configurable planner-policy values without making the default flow difficult to
  understand.
- [ ] Add deterministic validation for additional malformed task metadata.
- [ ] Evaluate whether different task collections require configurable policy profiles.

Planner changes should remain explainable and testable.

## Google completion bridge

Completing a `Kestrel Today` projection currently does not complete the canonical Nextcloud task.

A future completion bridge should be designed carefully because it crosses from derived state into
authoritative state.

- [ ] Define completion semantics for a Google projection.
- [ ] Determine whether Google completion should be treated as a proposal or direct user intent.
- [ ] Map the projection reliably back to its canonical Nextcloud UID.
- [ ] Add narrowly scoped Nextcloud completion capability.
- [ ] Prevent stale or duplicate projections from completing the wrong task.
- [ ] Record completion provenance.
- [ ] Preserve safe recovery behavior when either side is unavailable.

This capability should not be added merely as a generic synchronization rule.

## Recurring tasks and reminders

Recurring CalDAV tasks require additional identity semantics.

The current prototype intentionally excludes recurring reminder collections.

- [ ] Model `RECURRENCE-ID` and recurring VTODO identity correctly.
- [ ] Distinguish recurring task definitions from individual instances.
- [ ] Test completed and skipped recurrence instances.
- [ ] Prevent one recurrence instance from overwriting another.
- [ ] Add recurring collections only after canonical identity is reliable.

## Local LLM reasoning

Ollama is already part of the Kestrel stack, but the working planner does not yet depend on a model.

The first LLM milestone should augment deterministic policy rather than replace it.

Potential capabilities include:

- [ ] Select and document an initial local model.
- [ ] Define structured model input and output schemas.
- [ ] Explain why competing tasks deserve attention.
- [ ] Identify vague or oversized tasks.
- [ ] Suggest task decomposition.
- [ ] Estimate qualitative effort.
- [ ] Identify likely dependencies or blocked work.
- [ ] Suggest stale-backlog decisions.
- [ ] Combine planner output with calendar context.
- [ ] Reject malformed model output safely.
- [ ] Verify that planner operation continues safely when inference is unavailable.

Hard rules such as deadlines, future start dates, and authority boundaries should remain
deterministic.

## Controlled authoritative writes

Kestrel currently has no task-content write authority over canonical Nextcloud task collections.

Future writes should be added incrementally.

### Initial controlled-write milestone

- [ ] Add a deterministic policy gateway separate from the LLM.
- [ ] Allow an LLM or extractor to propose new tasks without immediate side effects.
- [ ] Define a narrow canonical task-creation connector.
- [ ] Require explicit human approval before authoritative creation.
- [ ] Restrict allowed destination task lists.
- [ ] Enforce configurable per-request and rate limits.
- [ ] Validate fields.
- [ ] Detect likely duplicates.
- [ ] Record source provenance.
- [ ] Record an audit trail.
- [ ] Keep arbitrary update, completion, and delete capabilities unavailable to the model.

Creation should be introduced before unrestricted mutation because it is easier to constrain,
review, and audit.

## Joplin integration

Joplin is a planned personal knowledge source.

Integration should use Joplin-aware tooling rather than attempting to parse its synchronization
storage directly.

- [ ] Establish a headless or API-based Joplin integration.
- [ ] Read approved notebooks and notes.
- [ ] Preserve note identity and provenance.
- [ ] Define optional explicit task/action syntax.
- [ ] Extract explicit action items deterministically where possible.
- [ ] Use the LLM for ambiguous candidate-action detection.
- [ ] Require approval before converting inferred actions into canonical tasks.
- [ ] Add note retrieval/search for broader personal context.

## Email integration

Email can provide valuable action context but is also an untrusted-input surface.

- [ ] Add Gmail ingestion with narrowly scoped permissions.
- [ ] Evaluate Microsoft/Outlook ingestion.
- [ ] Identify explicit and candidate action items.
- [ ] Preserve message provenance.
- [ ] Treat message content as untrusted data, not instructions.
- [ ] Require policy checks before any resulting task creation.
- [ ] Support summarization without granting unnecessary mail-write capabilities.
- [ ] Consider reply/draft capabilities only under a separate threat model.

## Personal retrieval and knowledge search

Longer-term Kestrel functionality should support grounded retrieval across approved personal
sources.

- [ ] Nextcloud document/file retrieval.
- [ ] Joplin note retrieval.
- [ ] Email retrieval.
- [ ] Cross-source indexing.
- [ ] Embedding/vector retrieval where useful.
- [ ] Keyword and metadata retrieval where appropriate.
- [ ] Source provenance and citations in generated answers.
- [ ] Retrieval filtering based on source permissions.
- [ ] Protection against retrieved-content prompt injection.
- [ ] Local RAG over approved personal context.

Vector search should be one retrieval method rather than an architectural requirement for every
source.

## Human-facing assistant

The orchestration, policy, storage, and reasoning layers should remain usable independently of a
specific chat interface.

- [ ] Evaluate candidate self-hosted assistant interfaces.
- [ ] Define the minimum interface capabilities Kestrel requires.
- [ ] Avoid coupling core Kestrel logic to one frontend.
- [ ] Provide task/planner explanations conversationally.
- [ ] Surface proposed authoritative actions clearly for approval.
- [ ] Provide provenance links where possible.
- [ ] Support useful passive delivery such as a morning planning digest.

## Audit and observability

As Kestrel gains more capabilities, operational visibility should grow with them.

- [ ] Define structured audit records.
- [ ] Record authoritative write proposals and approvals.
- [ ] Record executed authoritative writes.
- [ ] Track connector failures without leaking secrets.
- [ ] Expose enough planner diagnostics to explain surprising selections.
- [ ] Define retention policies for sensitive logs.
- [ ] Add health checks for major local services.
- [ ] Add useful failure notifications without creating notification noise.

## Longer-term vision

Potential later capabilities include:

- [ ] Personal knowledge organization and cleanup assistance.
- [ ] Cross-source reasoning with provenance and citations.
- [ ] Modular connectors for additional personal systems.
- [ ] Fine-grained user-configurable permission policies.
- [ ] Safe, reversible write capabilities where they provide meaningful value.
- [ ] Delegation/waiting workflows.
- [ ] Project and goal-level planning.
- [ ] Personal planning across longer horizons while retaining a focused daily view.

## Roadmap principles

New roadmap work should preserve these rules:

1. **Authoritative systems remain explicit.**
2. **Read and write authority should be separated wherever possible.**
3. **Deterministic code handles deterministic work.**
4. **LLMs provide judgment, not security boundaries.**
5. **Derived state should remain reconstructable.**
6. **Consequential writes should be narrow and auditable.**
7. **Features should fail safely when dependencies or configuration are unavailable.**
8. **A feature is not complete until its permission model is understood.**

Kestrel should grow by adding clearly bounded capabilities, not by gradually granting one assistant
unrestricted access to everything.
