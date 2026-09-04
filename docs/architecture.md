# Architecture

## Vision

Kestrel is intended to become a local-first assistant that can reason over personal operational and
knowledge systems without becoming an unrestricted autonomous agent.

The project should eventually support multiple classes of source data, including:

- tasks and reminders;
- calendars;
- notes and notebooks;
- email;
- documents and files;
- other user-approved personal context sources.

## Architectural direction

```text
Personal data sources
        |
        v
Connectors / deterministic ingestion
        |
        v
Normalized context and policy boundary
        |
        +------> local retrieval / search
        |
        v
Local LLM / reasoning layer
        |
        v
Recommendations and proposed actions
        |
        v
Human approval + policy enforcement
        |
        v
Narrow write-capable connectors
```

## Responsibilities

### Connectors

Connect to external systems using the minimum permissions required. Read and write capabilities
should be separable wherever possible.

### Deterministic automation

Use conventional software for reliable work such as synchronization, normalization, duplicate
detection, scheduling, and explicit task capture. Do not use an LLM when deterministic code can
perform the job more safely.

### Reasoning layer

Use local language models for tasks where judgment is useful, such as prioritization, summarization,
task decomposition, stale-item review, and identifying potential action items.

### Policy boundary

The model must not be the final authority on what it is allowed to do. A separate policy layer
should constrain side effects, validate requests, enforce quantity/rate limits, and maintain an
audit trail.

## Initial implementation direction

The initial task-planning milestone is expected to use:

- Docker Compose for reproducible deployment;
- a project-specific Ollama instance for local inference;
- Nextcloud Tasks as the initial authoritative task store;
- Nextcloud Calendar as the initial personal calendar source;
- an orchestration layer for deterministic integrations;
- a human-facing assistant interface selected independently from the core services.

Specific products beyond these initial decisions should remain replaceable where practical.
