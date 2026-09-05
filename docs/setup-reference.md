# Setup Reference

This document records the configuration requirements for the current Kestrel prototype.

It is intended to serve as an authoritative reference for both human installers and an LLM guiding
someone through setup.

The preferred installation experience is **not** to manually reconstruct the Activepieces flow.
Import the tested flow template and configure the deployment-specific values described here.

Canonical flow template:

[`flows/build-kestrel-today.json`](../flows/build-kestrel-today.json)

Planning behavior is documented separately in:

[`docs/planner-policy.md`](planner-policy.md)

## Current prototype scope

The current working prototype:

1. reads unfinished tasks from two Nextcloud Tasks collections;
2. normalizes their task metadata;
3. builds a deterministic daily plan;
4. projects the selected tasks into a Google Tasks list named `Kestrel Today`;
5. reconciles that projection hourly.

Nextcloud remains authoritative.

Google Tasks is currently a disposable, mobile-friendly projection surface.

The current flow does **not** write task content back to Nextcloud.

## Tested deployment baseline

The repository currently provides a Docker Compose deployment containing:

| Component | Current pinned image/version |
| --- | --- |
| Activepieces app | `ghcr.io/activepieces/activepieces:0.90.2` |
| Activepieces worker | `ghcr.io/activepieces/activepieces:0.90.2` |
| PostgreSQL | `pgvector/pgvector:0.8.0-pg14` |
| Redis | `redis:7.0.7` |
| Ollama | `ollama/ollama:0.32.4-rocm` |

These versions describe the currently tested baseline.

They should not be interpreted as permanent requirements unless later documentation says otherwise.

Ollama is present for later local-LLM functionality but is not required by the current deterministic
task planner.

## Activepieces deployment

The repository's `compose.yaml` separates the Activepieces app and worker.

The app is exposed to the user through:

```text
AP_FRONTEND_URL
```

The worker uses the Docker-internal Activepieces service address:

```text
http://activepieces
```

This avoids requiring the worker to resolve the browser-facing hostname in order to communicate with
the Activepieces app.

### Required `.env` values

Start from:

```text
.env.example
```

At minimum, configure the deployment values required by the Compose stack, including:

```text
AP_FRONTEND_URL
AP_POSTGRES_PASSWORD
AP_ENCRYPTION_KEY
AP_JWT_SECRET
AP_WORKER_TOKEN
```

Do not commit the resulting `.env` file.

### Activepieces execution policy

The tested deployment uses:

```text
AP_EXECUTION_MODE=SANDBOX_CODE_ONLY
AP_WORKER_CONCURRENCY=1
```

Kestrel should not require weakening the Activepieces code sandbox merely to make the current flow
work.

The flow performs network access through Activepieces pieces rather than through arbitrary network
calls from Code steps.

## Importing the Kestrel flow

Import:

```text
flows/build-kestrel-today.json
```

from the Activepieces flow dashboard.

Import it as a **new flow**.

Do not import it into an existing flow unless intentionally replacing that flow.

### Expected import behavior

The current template has been tested with Activepieces 0.90.2.

Expected behavior:

- the flow imports successfully;
- the imported flow is deactivated;
- Nextcloud references remain project-variable expressions;
- Google Tasks connection fields are empty;
- the installer selects their own Google Tasks connection;
- no source deployment credentials are embedded in the template.

The flow should remain deactivated until configuration and testing are complete.

## Activepieces project variables

The imported flow requires exactly six project variables:

```text
NEXTCLOUD_BASE_URL
NEXTCLOUD_USERNAME
NEXTCLOUD_APP_PASSWORD
NEXTCLOUD_PERSONAL_COLLECTION
NEXTCLOUD_CLASSES_COLLECTION
KESTREL_TIMEZONE
```

These are **Activepieces project variables**, not variables in the repository `.env` file.

### `NEXTCLOUD_BASE_URL`

The base URL of the user's Nextcloud instance.

Example:

```text
https://cloud.example.net
```

Do not include a trailing slash.

The flow constructs CalDAV URLs from this value.

### `NEXTCLOUD_USERNAME`

The Nextcloud username used by Kestrel.

A dedicated Kestrel service account is strongly preferred over the user's primary Nextcloud account.

Example:

```text
kestrel
```

The account should have only the access required for Kestrel's current task sources.

### `NEXTCLOUD_APP_PASSWORD`

A dedicated Nextcloud application password for the Kestrel account.

Treat this value as a secret.

Do not:

- commit it to Git;
- place it directly in the exported flow;
- paste it into shell commands where it may be retained in shell history;
- put it into documentation.

Store it in the Activepieces project variable.

### `NEXTCLOUD_PERSONAL_COLLECTION`

The CalDAV collection path segment corresponding to the task list Kestrel should treat as
**Personal**.

Example shape:

```text
personal_shared_by_username
```

This is the collection identifier used in the CalDAV URL, not necessarily the human-readable task
list title.

Do not include leading or trailing slashes.

### `NEXTCLOUD_CLASSES_COLLECTION`

The CalDAV collection path segment corresponding to the task list Kestrel should treat as
**Classes**.

Example shape:

```text
classes_shared_by_username
```

Again, use the CalDAV collection identifier rather than assuming the visible task-list name is the
correct path.

Do not include leading or trailing slashes.

### `KESTREL_TIMEZONE`

The IANA timezone Kestrel should use when interpreting planning dates.

Examples:

```text
America/New_York
America/Chicago
Europe/London
```

Do not use an arbitrary abbreviation such as:

```text
EST
EDT
CST
```

The flow validates the supplied timezone through JavaScript's `Intl.DateTimeFormat`.

An invalid timezone causes the planner to fail rather than silently using a different timezone.

## Nextcloud task-source model

The current planner expects two logical source collections:

- Personal
- Classes

The physical Nextcloud collection names are configured through the project variables above.

### Recommended account model

Use a dedicated Nextcloud account for Kestrel.

The primary user should share only the required task lists with that account.

For the current prototype, those shares should be **read-only**.

This creates a server-enforced boundary between:

```text
Kestrel can read canonical tasks
```

and:

```text
Kestrel can modify canonical tasks
```

The second capability is intentionally absent.

### Why read-only sharing matters

The Activepieces flow currently sends CalDAV `REPORT` requests to retrieve unfinished VTODO
components.

The flow does not need Nextcloud task-write permission to perform its current job.

Read-only access therefore limits the consequences of:

- a flow bug;
- an incorrect planner decision;
- accidental future changes to the Activepieces flow;
- model errors once LLM functionality is introduced.

Kestrel's lack of canonical write authority should be enforced by Nextcloud permissions, not merely
by convention inside the flow.

### Required task metadata

The current CalDAV query requests these VTODO fields:

```text
UID
SUMMARY
DESCRIPTION
CREATED
DTSTART
DUE
PRIORITY
STATUS
PERCENT-COMPLETE
RELATED-TO
```

The flow also reads the CalDAV ETag for each result.

Not every task must populate every optional field.

The planner is designed to handle missing dates and unspecified priority.

### Completed tasks

The current CalDAV query selects tasks for which the `COMPLETED` property is not defined.

Completed Nextcloud tasks therefore do not enter the current planning pipeline.

### Recurring tasks

Recurring-task collections are not currently part of the supported prototype.

CalDAV recurrence can represent multiple task instances using the same UID plus recurrence metadata,
which requires more careful canonical identity handling than the current simple task model.

Do not add recurring-reminder collections to the current flow merely by duplicating the Personal or
Classes read step.

## Finding the CalDAV collection identifiers

The required collection values are the path components used by Nextcloud's CalDAV endpoint.

The flow ultimately constructs URLs in this form:

```text
<NEXTCLOUD_BASE_URL>/remote.php/dav/calendars/<NEXTCLOUD_USERNAME>/<COLLECTION>/
```

For example:

```text
https://cloud.example.net/remote.php/dav/calendars/kestrel/personal_shared_by_user/
```

The exact collection identifier depends on the Nextcloud installation and how the task list was
created or shared.

A guided setup assistant should help the installer discover these values from their own Nextcloud
CalDAV resources rather than guessing them.

Do not assume that the visible list title can simply be lowercased and used as the collection path.

## Google Tasks configuration

The current prototype requires a Google account accessible through the Activepieces Google Tasks
piece.

### Required list

Create exactly one Google Tasks list named:

```text
Kestrel Today
```

The name is currently part of the Kestrel interface contract.

The flow discovers the Google list ID dynamically each time it runs.

The Google list ID is therefore **not** a setup variable and should not be hardcoded into the
template.

### Duplicate list names

Kestrel requires exactly one list whose trimmed title is:

```text
Kestrel Today
```

If no matching list exists, the flow fails with an explanatory error.

If multiple matching lists exist, the flow also fails rather than guessing which one to use.

### Google Tasks connection

After importing the flow, select a Google Tasks connection for every Google Tasks action that has an
empty Connection field.

The same connection should normally be used for all of them.

Current Google-connected actions are:

```text
List Google Task Lists
Read Kestrel Today
Add Projected Task
Update Projected Task
Delete Projected Task
Read Kestrel Today After Reconcile
Move Projected Task
```

The public flow template intentionally contains no deployment-specific Activepieces connection ID.

### What Kestrel may modify in Google

The current flow may:

- create Kestrel-managed projected tasks;
- update their title, notes, and due date;
- remove obsolete or duplicate active projections;
- reorder active tasks so the Kestrel projection appears in planner order.

This broad write access is acceptable because `Kestrel Today` is a derived projection rather than
the canonical task store.

### Managed-task marker

Projected Google tasks include:

```text
Kestrel daily projection
```

in their notes.

They also include a line in this form:

```text
Kestrel source UID: <Nextcloud UID>
```

Kestrel uses these markers to distinguish its own projections from unrelated Google tasks.

Unmanaged Google tasks are ignored by reconciliation.

Do not manually add the Kestrel marker to unrelated Google tasks unless intentionally placing them
under Kestrel management.

## First-run validation

Keep the flow deactivated while configuring it.

A recommended validation sequence is:

1. verify the six Activepieces project variables;
2. test `Read Personal Tasks`;
3. test `Read Classes Tasks`;
4. confirm both return CalDAV `207 Multi-Status`;
5. test `Normalize Nextcloud Tasks`;
6. inspect Personal and Classes task counts;
7. test `Build Candidate Today`;
8. inspect the selected tasks and reasons;
9. configure the Google Tasks connection on all seven Google actions;
10. verify exactly one `Kestrel Today` list exists;
11. run the complete flow manually;
12. inspect the resulting Google Tasks list;
13. run the flow a second time.

The second run should converge.

Under stable source data, reconciliation should report:

```text
toCreate: 0
toUpdate: 0
toDelete: 0
```

The exact number of unchanged tasks is not fixed because the planner's candidate set depends on the
user's current tasks.

## What to verify in Google Tasks

After a successful run:

- selected Kestrel tasks should appear near the top of `Kestrel Today`;
- their order should match the planner's rank;
- source due dates should be preserved;
- undated Nextcloud tasks should remain undated in Google;
- backlog-review items should be prefixed with `Review:`;
- projected-task notes should explain selection role, priority, reason, score, source, and canonical
  UID;
- unrelated Google tasks should remain untouched.

## Enabling the flow

Only enable/publish the scheduled flow after manual testing succeeds.

The current trigger runs:

```text
Every Hour
```

including weekends.

Hourly execution is intended to keep the mobile projection reasonably current while avoiding
unnecessary high-frequency polling.

## Expected failure behavior

Kestrel should prefer clear failure over silently operating on ambiguous configuration.

Examples of intentional failure cases include:

- missing or invalid timezone;
- missing `Kestrel Today` list;
- duplicate `Kestrel Today` lists;
- missing Google authentication;
- malformed planning date;
- missing Google task ID during update;
- incomplete task resolution during ordering.

A setup guide or LLM assistant should diagnose the failing step rather than bypassing these checks.

## Configuration that should not be personalized inside the flow

Do not replace portable expressions in the public template with deployment-specific values.

The public template should not contain:

- Nextcloud hostname;
- personal Nextcloud username;
- Nextcloud app password;
- CalDAV collection names;
- Google Tasks list ID;
- Activepieces Google connection ID;
- user-specific timezone.

These belong in project variables, dynamically resolved resources, or connection configuration.

## Secret-handling rules

During setup:

- never commit secrets;
- prefer application-specific credentials over primary passwords;
- do not paste secrets into an LLM conversation;
- avoid putting secrets directly into shell commands;
- use Activepieces variables/connections for runtime credentials;
- sanitize exported flows before publication;
- inspect Git changes before committing them.

The repository's secret-scanning controls are a secondary safety net, not a substitute for careful
secret handling.

## Canonical implementation versus documentation

The flow JSON is the canonical executable implementation.

This document explains the configuration contract around it.

If implementation and documentation disagree:

1. investigate the discrepancy;
2. determine whether the flow or the documented contract is wrong;
3. update them together.

Do not manually reconstruct the flow from prose documentation as the normal installation path.

## Guided setup

The repository provides:

[`guided-setup-prompt.md`](guided-setup-prompt.md)

The guided setup prompt uses this document as its configuration reference and instructs the setup
assistant to:

- proceed one logical step at a time;
- wait for the user to report the result of each important step;
- adapt to the actual Activepieces and Nextcloud interfaces encountered;
- never invent UI controls that the user has not confirmed exist;
- request screenshots or visible options when interfaces differ;
- never request secret values in chat;
- explain unfamiliar configuration rather than merely issuing commands;
- preserve Kestrel's least-privilege boundaries;
- use the imported flow instead of asking the installer to rebuild it manually.

The goal is reproducibility with assistance, not a brittle click-by-click manual tied to one exact
version of every UI.
