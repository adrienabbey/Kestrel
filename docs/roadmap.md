# Roadmap

This roadmap intentionally separates a useful first system from broader long-term goals. It does not
assign artificial calendar dates to work that has no external deadline.

## Near-term MVP

- [ ] Establish repository hygiene, security documentation, and secret scanning.
- [ ] Create a reproducible Docker Compose development stack.
- [ ] Run an isolated local Ollama service for Kestrel.
- [ ] Connect to Nextcloud Tasks with read-only access first.
- [ ] Connect to personal Nextcloud calendars for workload/context awareness.
- [ ] Define task metadata and stale-task prioritization behavior.
- [ ] Produce a read-only daily recommendation from current tasks and calendar load.
- [ ] Add deterministic capture/import from Google Tasks into the authoritative task system.
- [ ] Evaluate the human-facing assistant interface without coupling core logic to it.

## Controlled write milestone

- [ ] Add a policy gateway separate from the LLM.
- [ ] Allow the model to propose new tasks without side effects.
- [ ] Require explicit human approval before creation.
- [ ] Enforce configurable per-request and rate limits.
- [ ] Validate fields and detect likely duplicates.
- [ ] Record an audit trail and source provenance.
- [ ] Do not expose update, completion, or delete operations.

## Later integrations

- [ ] Gmail ingestion and action-item suggestions.
- [ ] Microsoft/Outlook ingestion and action-item suggestions.
- [ ] Joplin synchronization through a Joplin-aware client/API rather than raw sync-file parsing.
- [ ] Explicit Joplin task/action syntax and deterministic extraction.
- [ ] Read/search access to Joplin notes for discussion and organization.
- [ ] Nextcloud document/file retrieval.
- [ ] Cross-source personal search and RAG.
- [ ] Candidate task extraction from notes and email with human approval.
- [ ] Morning digest or other passive recommendation delivery.
- [ ] Stale-backlog review workflows: do, break down, defer, delegate/wait, or remove.
- [ ] Revisit a reliable Android client for Nextcloud VTODO tasks.

## Longer-term vision

- [ ] Personal knowledge organization and cleanup assistance.
- [ ] Cross-source provenance and citation support.
- [ ] Modular connectors for additional personal systems.
- [ ] Fine-grained user-configurable permission policies.
- [ ] Safe, reversible write capabilities where they provide meaningful value.
