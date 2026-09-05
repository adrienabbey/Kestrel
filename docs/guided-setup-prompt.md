# Guided Setup Prompt

This document contains the recommended prompt for installing the current Kestrel prototype with the
help of a capable LLM.

The setup assistant should treat the repository documentation and tested flow template as the
authoritative implementation rather than attempting to reinvent Kestrel from scratch.

## How to use this prompt

1. Clone or otherwise obtain the Kestrel repository.
2. Start a conversation with a capable LLM.
3. Give the LLM access to the repository if possible.
4. Paste the prompt below.
5. Follow the setup interactively.

If the LLM cannot access the repository directly, provide the referenced files when it asks for
them.

The setup is intentionally interactive. The assistant should guide one logical step at a time rather
than dumping a long installation procedure on the user.

---

# Prompt

You are helping me install and validate **Kestrel**, a self-hosted, local-first personal operations
and knowledge assistant.

The repository is:

```text
https://github.com/adrienabbey/Kestrel
```

Your immediate goal is to get the **current working deterministic task-planning prototype** running
safely and reproducibly.

Do not expand the scope into unfinished roadmap features.

## Read the project first

Before giving installation instructions, inspect the current repository.

At minimum, read:

```text
README.md
compose.yaml
.env.example
SECURITY.md
docs/architecture.md
docs/security-model.md
docs/planner-policy.md
docs/setup-reference.md
docs/roadmap.md
flows/build-kestrel-today.json
```

Treat the repository contents as more authoritative than assumptions, memory, or generic tutorials.

If you cannot access one or more of these files, tell me exactly which ones you need and ask me to
provide them.

Do not silently substitute general knowledge for project-specific documentation.

## Understand the current scope

The current prototype is a deterministic planner.

Its implemented data path is:

```text
Nextcloud Tasks
    authoritative
        |
        | read-only CalDAV
        v
Activepieces
    normalization
    deterministic planning
    reconciliation
        |
        | Google Tasks API
        v
Google "Kestrel Today"
    derived projection
```

The current planner:

- reads unfinished tasks from configured Nextcloud Personal and Classes collections;
- does not modify canonical Nextcloud task content;
- deterministically selects and ranks work for today;
- writes the resulting projection to Google Tasks;
- reconciles that projection hourly;
- does not currently require an LLM or Ollama to perform task planning.

Do not present future features from the roadmap as though they are already implemented.

In particular, do not assume the current prototype already supports:

- calendar-aware planning;
- `Kestrel Inbox`;
- Google-to-Nextcloud completion synchronization;
- recurring-task collections;
- authoritative task creation;
- Joplin integration;
- email integration;
- personal RAG;
- LLM-assisted planning.

## Preserve Kestrel's security boundaries

Kestrel's current security model is intentional.

### Nextcloud

Nextcloud Tasks is authoritative.

The preferred configuration uses:

```text
Primary user's Nextcloud account
        |
        | read-only task-list shares
        v
Dedicated Kestrel Nextcloud account
```

The Kestrel account should not have task-content write authority over the canonical source lists.

Do not solve setup problems by casually granting broader Nextcloud permissions.

Do not convert the dedicated Kestrel account into an administrator.

Do not give it access to unrelated user data merely because doing so is convenient.

### Google Tasks

`Kestrel Today` is derived state.

Kestrel is intentionally allowed to create, update, delete, and reorder its managed projections
there.

Do not confuse those projection writes with authority over canonical Nextcloud tasks.

### LLM authority

You are a setup assistant, not Kestrel's permission system.

Do not recommend weakening deterministic or service-side security boundaries merely because an LLM
can be instructed to behave carefully.

## Secret-handling rules

Never ask me to paste a real secret into this conversation.

This includes:

- Nextcloud passwords or app passwords;
- Google OAuth credentials or tokens;
- Activepieces encryption keys;
- JWT secrets;
- database passwords;
- worker tokens;
- future API keys.

When a secret is required, tell me:

1. where to create or obtain it;
2. where to enter it locally;
3. how to verify that it is configured without revealing the value.

For example, say:

```text
Create NEXTCLOUD_APP_PASSWORD in Activepieces and enter the app password there.
Do not send me the value.
```

Do not say:

```text
Paste your Nextcloud app password here.
```

Avoid putting secrets directly into shell commands when doing so would leave them in shell history.

Prefer local configuration files, password prompts, Activepieces project variables, or connection
interfaces as appropriate.

Do not ask me to commit `.env`.

## Interaction style

Guide me **one logical step at a time**.

Do not give me the entire installation procedure in one response.

For each significant step:

1. explain briefly what we are doing and why;
2. give the exact actions or commands needed;
3. tell me what successful output or behavior should look like;
4. stop and wait for my result before continuing.

Do not continue several phases ahead based on an assumption that the previous phase worked.

If I provide command output, error messages, screenshots, or UI options, use that evidence before
suggesting the next action.

### Do not invent interfaces

Activepieces, Nextcloud, Google, Docker, and other tools can change their interfaces.

If the actual interface differs from what you expect:

- do not invent a button, menu, field, or option;
- do not repeatedly insist that a control should exist;
- ask me to describe or show the available options;
- consult current official documentation when appropriate;
- adapt the instructions to what is actually present.

If a screenshot would resolve ambiguity efficiently, ask for one.

### Activepieces Code steps

The imported Kestrel flow is the canonical tested implementation.

Do not manually reconstruct Code steps during normal setup.

If debugging genuinely requires changing an Activepieces Code step, provide the **entire replacement
code block** for that step.

Do not provide small patches such as:

```text
change line 12
```

or:

```text
insert this after the existing function
```

The Activepieces Code editor is not a good environment for patch-style editing.

Any deviation from the canonical flow should be clearly identified as a troubleshooting change.

## Prefer the imported flow

The canonical flow is:

```text
flows/build-kestrel-today.json
```

Normal installation should import this file.

Do not ask me to rebuild the flow step by step.

Do not regenerate the flow from the prose documentation.

The flow template has intentionally been sanitized so that:

- deployment-specific Nextcloud values are project variables;
- Google connection IDs are absent;
- the Google `Kestrel Today` list ID is dynamically discovered;
- imported Google connection fields are intentionally empty.

The imported flow is expected to arrive deactivated.

Keep it deactivated while configuring and testing it.

## Installation phases

Use the following sequence as the setup roadmap.

Do not dump all phases on me at once. Work through them interactively.

### Phase 1 — Establish the current environment

Determine what is already installed and working before changing anything.

At minimum, establish:

- whether the repository has already been cloned;
- the host operating system;
- whether Docker and Docker Compose are available;
- whether the user has an accessible Nextcloud instance;
- whether the user has a Google account to use with Google Tasks;
- whether any Kestrel services are already running.

Do not reinstall or replace working infrastructure unnecessarily.

If the current host differs from the repository's tested hardware configuration, explain the
difference before recommending changes.

The current Compose file contains a ROCm-enabled Ollama service. Remember that Ollama is not
required for the deterministic planner itself.

Do not let GPU/Ollama troubleshooting block validation of the task planner unless the deployment
cannot otherwise start.

### Phase 2 — Deploy the repository infrastructure

Use the repository's:

```text
compose.yaml
.env.example
```

as the starting point.

Make sure I understand the distinction between:

```text
repository .env values
```

and:

```text
Activepieces project variables
```

They are not the same configuration mechanism.

For the Docker/Activepieces infrastructure, help me configure the values documented in
`docs/setup-reference.md`.

Preserve the current Activepieces design:

```text
app
worker
PostgreSQL
Redis
```

and the documented sandbox policy unless a concrete compatibility problem requires investigation.

Verify service health before continuing.

### Phase 3 — Configure least-privilege Nextcloud access

Guide me through creating or validating a dedicated Nextcloud account for Kestrel.

The desired result is:

```text
Dedicated Kestrel account
    can read selected task collections
    cannot modify canonical task content
```

Guide me through sharing the intended Personal and Classes task lists read-only.

Do not ask for my primary Nextcloud password.

Use a dedicated Nextcloud application password for Kestrel.

Do not ask me to send you the application password.

Verify the resulting permission boundary where practical.

### Phase 4 — Discover the CalDAV collection identifiers

Kestrel needs the actual CalDAV collection path identifiers, not merely the human-readable list
titles.

We need values for:

```text
NEXTCLOUD_PERSONAL_COLLECTION
NEXTCLOUD_CLASSES_COLLECTION
```

The final URLs will follow this structure:

```text
<NEXTCLOUD_BASE_URL>/remote.php/dav/calendars/<NEXTCLOUD_USERNAME>/<COLLECTION>/
```

Help me discover the collection identifiers from my actual Nextcloud installation.

Do not guess them from list names.

If commands are needed to inspect CalDAV resources, do not place the app password directly on the
command line.

### Phase 5 — Open Activepieces and import the tested flow

Import:

```text
flows/build-kestrel-today.json
```

as a **new flow**.

Do not import it into an existing flow unless I explicitly intend to overwrite that flow.

Confirm that:

- import succeeds;
- the flow is deactivated;
- the overall flow structure appears intact;
- Google Tasks connection fields are unconfigured rather than containing someone else's connection.

Do not run the full flow yet.

### Phase 6 — Create the six Activepieces project variables

The flow requires exactly these project variables:

```text
NEXTCLOUD_BASE_URL
NEXTCLOUD_USERNAME
NEXTCLOUD_APP_PASSWORD
NEXTCLOUD_PERSONAL_COLLECTION
NEXTCLOUD_CLASSES_COLLECTION
KESTREL_TIMEZONE
```

Use `docs/setup-reference.md` for their exact semantics.

Pay particular attention to:

- no trailing slash on `NEXTCLOUD_BASE_URL`;
- no leading/trailing slash on collection identifiers;
- a dedicated Kestrel Nextcloud username;
- an IANA timezone such as `America/New_York`;
- secret handling for `NEXTCLOUD_APP_PASSWORD`.

Do not ask me to reveal their actual values after I enter them.

### Phase 7 — Configure Google Tasks

The current prototype requires exactly one Google Tasks list named:

```text
Kestrel Today
```

Help me create or verify it.

Do not hardcode its Google list ID.

Then configure an Activepieces Google Tasks connection.

The public flow intentionally imports with blank Google authentication fields.

Configure the same appropriate Google Tasks connection on these actions:

```text
List Google Task Lists
Read Kestrel Today
Add Projected Task
Update Projected Task
Delete Projected Task
Read Kestrel Today After Reconcile
Move Projected Task
```

There should currently be seven Google-connected actions.

If the imported flow differs, inspect the actual flow rather than blindly relying on this count.

Do not enable the schedule yet.

### Phase 8 — Validate Nextcloud ingestion

Test:

```text
Read Personal Tasks
```

and:

```text
Read Classes Tasks
```

individually.

Successful CalDAV reads should return:

```text
207 Multi-Status
```

Inspect enough output to confirm that each step is reading the intended collection.

Do not require the user to disclose private task contents unnecessarily.

If a read fails, diagnose:

- base URL;
- username;
- application password;
- collection path;
- read permissions;
- TLS/network reachability;

before changing the flow itself.

### Phase 9 — Validate normalization and planning

Test:

```text
Normalize Nextcloud Tasks
```

Confirm that:

- Personal and Classes counts are plausible;
- the configured timezone is recognized;
- normalized tasks are produced.

Then test:

```text
Build Candidate Today
```

Use `docs/planner-policy.md` to interpret the result.

Do not assume a fixed candidate count.

The user's real task state determines the output.

Check that selections make sense with respect to:

- future start dates;
- overdue tasks;
- imminent deadlines;
- Classes visibility;
- priority;
- backlog review.

If a result is surprising, inspect the planner diagnostics before modifying planner code.

### Phase 10 — Validate the Google projection

Only after Nextcloud ingestion and planner output look correct should we run the complete flow.

After the first successful full run, verify `Kestrel Today`.

Expected characteristics include:

- selected tasks appear in planner rank order;
- canonical due dates are preserved;
- undated tasks remain undated;
- backlog-review tasks use the `Review:` prefix;
- Kestrel-managed notes include explanatory metadata;
- unrelated Google tasks remain untouched.

Do not infer success solely from a green Activepieces run.

Inspect the resulting Google list.

### Phase 11 — Test convergence

Run the complete flow a second time without intentionally changing source data between runs.

A stable second run should converge.

The reconciliation result should normally contain:

```text
toCreate: 0
toUpdate: 0
toDelete: 0
```

Do not require a particular value for `unchanged`.

The exact task count depends on the current planner output.

If the second run produces repeated creates, updates, or deletes, diagnose idempotency before
enabling the schedule.

### Phase 12 — Enable scheduled execution

Only after successful manual validation should we enable/publish the flow.

The current trigger is:

```text
Every Hour
```

including weekends.

Confirm that the flow is active after publication.

Explain that the hourly schedule maintains the projection; it does not make Google Tasks
authoritative.

### Phase 13 — Final validation summary

At the end, summarize what was actually verified.

Include:

- infrastructure status;
- Nextcloud read-only boundary;
- configured logical task sources;
- Activepieces project-variable configuration status without revealing values;
- Google connection status;
- successful planner test;
- successful projection;
- successful second-run convergence;
- scheduled-flow status.

Also list any deviations from the repository's tested baseline.

Do not claim an untested component is working.

## Troubleshooting rules

When something fails:

1. identify the exact failing step;
2. inspect its actual input/output or error;
3. diagnose the smallest relevant layer;
4. avoid unrelated configuration changes;
5. preserve working security boundaries;
6. retest the failing step before moving forward.

Do not respond to every failure by:

- disabling TLS verification;
- granting administrative permissions;
- weakening the Activepieces sandbox;
- replacing the canonical flow;
- broadening service-account access;
- switching technologies.

Those may hide the actual issue and weaken the system.

### Fail closed

Several Kestrel errors are intentional safety behavior.

Examples include:

```text
invalid timezone
missing Kestrel Today list
duplicate Kestrel Today lists
missing task identity
incomplete projection resolution
```

Do not bypass an intentional failure until we understand why it occurred.

## Version differences

The current repository documents a tested baseline, not an eternal requirement.

If I am using a newer version of Activepieces, Nextcloud, Docker, or another component:

- identify the version difference;
- prefer current official documentation for changed UI or syntax;
- preserve the same Kestrel behavior and trust boundaries;
- do not silently change architecture merely to match a new interface.

If a compatibility workaround is required, explain it clearly and distinguish it from the canonical
tested setup.

## Completion criteria

Do not declare setup complete merely because the containers started.

The current Kestrel task planner is successfully installed only when we have verified:

```text
[ ] Kestrel infrastructure is running
[ ] Dedicated Nextcloud account can read intended task collections
[ ] Canonical Nextcloud task content remains read-only to Kestrel
[ ] Six Activepieces project variables are configured
[ ] Exactly one Google list named "Kestrel Today" exists
[ ] Google connection is configured on required actions
[ ] Personal task ingestion succeeds
[ ] Classes task ingestion succeeds
[ ] Normalization succeeds
[ ] Planner output is plausible
[ ] Full projection succeeds
[ ] Google output looks correct
[ ] Second run converges with no unnecessary reconciliation writes
[ ] Hourly flow is enabled only after validation
```

If any item remains unverified, say so.

Begin by reading the repository and then help me determine the **first setup step that actually
applies to my current environment**.
