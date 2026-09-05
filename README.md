# Kestrel

**Kestrel** is a self-hosted, local-first personal operations and knowledge assistant.

The long-term goal is to provide a privacy-conscious system that can reason across personal tasks,
calendars, notes, email, and documents while keeping the user in control of consequential actions.

Kestrel is being built around a simple principle:

> **Understand broadly, act narrowly.**

The current working prototype is a deterministic daily task planner built around Nextcloud Tasks,
Activepieces, and Google Tasks.

## Current status

Kestrel is **early experimental software**, but the first end-to-end task-planning workflow is
working.

Today, Kestrel can:

- read approved Nextcloud Tasks collections through CalDAV;
- keep canonical Nextcloud task content behind a read-only service-account boundary;
- normalize task metadata such as start date, due date, priority, and age;
- build a deterministic ranked daily plan;
- give academic tasks additional deadline visibility;
- surface overdue and imminent work automatically;
- periodically surface stale undated backlog items for review;
- project the daily plan into a Google Tasks list named `Kestrel Today`;
- preserve task provenance and selection reasoning in projection metadata;
- reconcile the Google projection idempotently;
- run the projection workflow hourly.

The current planner **does not use an LLM**.

Ollama is included in the local stack for future reasoning capabilities, but the working daily
planner continues to function without model inference.

## Current architecture

```text
                 AUTHORITATIVE
                 Nextcloud Tasks
                Personal + Classes
                         |
                         | read-only CalDAV
                         v
                  Activepieces
             normalize / plan / reconcile
                         |
                         | Google Tasks API
                         v
                 "Kestrel Today"
                 derived projection
```

The distinction between authoritative and derived data is intentional.

```text
Nextcloud Tasks
    = canonical task state

Google "Kestrel Today"
    = rebuildable daily projection
```

Kestrel currently has read access to canonical task content and broader write access only to the
derived Google projection.

See [`docs/architecture.md`](docs/architecture.md) for the full architecture.

## Project principles

- **Local-first:** Prefer local inference and self-hosted services where practical.
- **Human-controlled:** Recommendations may be broad; consequential side effects should be narrow
  and explicitly authorized.
- **Least privilege:** Components receive only the permissions required for their role.
- **Deterministic where possible:** Use conventional software for synchronization, normalization,
  reconciliation, and hard policy rules.
- **Open standards:** Prefer interoperable protocols and portable formats over unnecessary lock-in.
- **Auditable automation:** Preserve source identity, reasoning, and provenance where practical.
- **Graceful failure:** Failed models, integrations, or configuration should stop at safe boundaries
  rather than damaging authoritative data.

## Daily planner

The current planner uses deterministic rules rather than model judgment.

Examples include:

- future start dates suppress tasks until they become available;
- overdue work is always included;
- general tasks receive a short mandatory deadline window;
- Class tasks receive a larger deadline window;
- high-priority academic work can receive earlier attention;
- remaining daily capacity is filled using a deterministic score;
- old undated tasks can rotate into a backlog-review slot.

A normal day targets approximately six actionable tasks, but mandatory deadline work is never hidden
merely to satisfy that target.

Detailed policy, scoring, and ranking behavior are documented in:

[`docs/planner-policy.md`](docs/planner-policy.md)

## Repository structure

```text
Kestrel/
├── docs/
│   ├── architecture.md
│   ├── guided-setup-prompt.md
│   ├── planner-policy.md
│   ├── roadmap.md
│   ├── security-model.md
│   └── setup-reference.md
├── flows/
│   └── build-kestrel-today.json
├── .env.example
├── compose.yaml
├── README.md
├── SECURITY.md
└── ...
```

The canonical executable implementation of the current planner is:

[`flows/build-kestrel-today.json`](flows/build-kestrel-today.json)

The flow is intended to be **imported**, not reconstructed manually from documentation.

## Recommended setup

Kestrel is currently intended for technically competent self-hosters.

The recommended installation path is **LLM-guided setup** rather than a brittle click-by-click
manual.

Start with:

[`docs/guided-setup-prompt.md`](docs/guided-setup-prompt.md)

That prompt instructs a capable LLM to:

- read the repository before giving instructions;
- guide setup one logical step at a time;
- preserve Kestrel's security boundaries;
- avoid requesting secrets in chat;
- import the tested Activepieces flow rather than rebuilding it;
- adapt to the actual UI and software versions encountered;
- validate each stage before continuing;
- verify second-run reconciliation before enabling the hourly schedule.

For the underlying configuration contract, see:

[`docs/setup-reference.md`](docs/setup-reference.md)

## Tested deployment baseline

The current Docker Compose stack includes:

- Activepieces app;
- Activepieces worker;
- PostgreSQL / pgvector;
- Redis;
- Ollama.

The repository currently pins the versions used by the working prototype.

These represent a tested baseline rather than a promise that Kestrel will permanently require those
exact versions.

The current planner itself depends on Activepieces, Nextcloud, and Google Tasks.

Ollama and vector functionality are present for future capabilities but are not yet in the planning
path.

## Required external services

The current prototype expects access to:

### Nextcloud

Nextcloud Tasks is the canonical task store.

The recommended model uses:

```text
Primary user
    |
    | read-only shares
    v
Dedicated Kestrel Nextcloud account
```

The Kestrel account should receive access only to the task collections needed by the planner.

### Google Tasks

Create exactly one Google Tasks list named:

```text
Kestrel Today
```

Kestrel dynamically resolves that list rather than storing a user-specific Google list ID.

The list is a derived working surface and should not contain irreplaceable canonical data.

## Security model

Kestrel's security boundaries are part of the architecture, not an afterthought.

The current prototype intentionally does **not** possess task-content write authority over canonical
Nextcloud task collections.

Google projection writes are permitted because `Kestrel Today` is rebuildable derived state.

Future authoritative writes should be introduced only through narrow, deterministic policy controls
with explicit approval where appropriate.

Important principles include:

- service-side least privilege;
- no secrets in Git;
- dedicated application credentials;
- untrusted treatment of retrieved content;
- LLM output as proposal rather than authority;
- deterministic policy outside the model.

See:

[`docs/security-model.md`](docs/security-model.md)

For vulnerability reporting and deployment cautions, see:

[`SECURITY.md`](SECURITY.md)

## Configuration

The repository `.env` configures the local infrastructure.

Start from:

```text
.env.example
```

Do not commit the resulting `.env`.

The imported Activepieces flow separately requires six **Activepieces project variables**:

```text
NEXTCLOUD_BASE_URL
NEXTCLOUD_USERNAME
NEXTCLOUD_APP_PASSWORD
NEXTCLOUD_PERSONAL_COLLECTION
NEXTCLOUD_CLASSES_COLLECTION
KESTREL_TIMEZONE
```

These are not repository environment variables.

See [`docs/setup-reference.md`](docs/setup-reference.md) before configuring them.

## What is not implemented yet

The current prototype does not yet provide:

- calendar-aware workload planning;
- Google `Kestrel Inbox` capture;
- Google-to-Nextcloud completion synchronization;
- recurring-task normalization;
- canonical task creation;
- Joplin integration;
- email ingestion;
- document retrieval;
- personal RAG/search;
- LLM-assisted planner judgment;
- policy-gated authoritative writes.

These are roadmap items rather than hidden or partially documented features.

See:

[`docs/roadmap.md`](docs/roadmap.md)

## Development direction

Kestrel is intended to grow through clearly bounded capabilities.

The longer-term architecture separates:

```text
Source systems
    own canonical data

Deterministic connectors
    move and normalize data

Deterministic policy
    enforces hard rules

Local models
    provide judgment

Humans
    retain authority over consequential actions

Narrow connectors
    perform explicitly permitted side effects
```

Future integrations may include:

- calendars;
- Joplin;
- email;
- Nextcloud files;
- personal search and retrieval;
- local LLM planning assistance;
- controlled task creation;
- cross-source reasoning with provenance and citations.

The goal is not to gradually grant one autonomous agent unrestricted access to everything.

## Documentation

| Document | Purpose |
| --- | --- |
| [`docs/architecture.md`](docs/architecture.md) | Current and future system architecture |
| [`docs/security-model.md`](docs/security-model.md) | Trust boundaries, permissions, and threat model |
| [`docs/planner-policy.md`](docs/planner-policy.md) | Deterministic planner behavior and scoring |
| [`docs/setup-reference.md`](docs/setup-reference.md) | Installation and configuration contract |
| [`docs/guided-setup-prompt.md`](docs/guided-setup-prompt.md) | Recommended interactive installation prompt |
| [`docs/roadmap.md`](docs/roadmap.md) | Completed work and planned milestones |
| [`SECURITY.md`](SECURITY.md) | Security reporting and deployment policy |

## Experimental software

Kestrel currently handles personal operational data and should be treated accordingly.

Until the project documents otherwise:

- deploy it only in environments you trust;
- do not expose development services directly to the public Internet;
- use dedicated, least-privilege credentials;
- inspect permissions before connecting new personal-data sources;
- do not grant an LLM destructive authority over source systems;
- keep real secrets out of the repository.

## AI-assisted development

Kestrel has been developed with substantial assistance from generative AI, primarily ChatGPT by
OpenAI.

The project concept, requirements, architectural direction, security boundaries, task semantics,
testing, integration, and final project decisions are directed by Adrien Abbey. ChatGPT has been
used extensively to draft source code, configuration, documentation, troubleshooting steps, and
design proposals.

Much of that generated material has been incorporated after iterative testing and discussion rather
than being independently rewritten line-by-line. Accordingly, the repository should not be
interpreted as code and prose authored entirely by hand.

Adrien Abbey is responsible for the project's direction, integration, testing, maintenance, and
decisions about what generated material is accepted into the repository.

See [`docs/ai-usage.md`](docs/ai-usage.md) for additional details.

## License

Kestrel is licensed under the [MIT License](LICENSE).
