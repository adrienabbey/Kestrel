# Kestrel

**Kestrel** is a self-hosted, local-first personal operations and knowledge assistant.

The long-term goal is to provide a single, privacy-conscious interface for reasoning across personal
tasks, calendars, notes, email, and documents while keeping the user in control of any action that
changes data.

## Project principles

- **Local-first:** Prefer local inference and self-hosted services where practical.
- **Human-controlled:** Recommendations may be broad; side effects should be narrow and explicitly
  authorized.
- **Least privilege:** Components receive only the permissions they need.
- **Open standards:** Prefer interoperable protocols and portable data formats over vendor lock-in.
- **Auditable automation:** Important actions should be understandable, constrained, and logged.
- **Graceful failure:** A failed model or integration must not damage the user's source data.

## Initial scope

The first milestone focuses on task management and daily planning:

- use Nextcloud Tasks as the initial task source of truth;
- consider calendar load when recommending what to work on;
- surface neglected, non-urgent work instead of allowing it to disappear indefinitely;
- consolidate capture from secondary task sources;
- keep the first planner read-only;
- add task creation later behind explicit approval and hard policy limits.

Future integrations may include Joplin, Nextcloud files, email, document retrieval, and broader
personal knowledge management.

## Status

Kestrel is **early experimental software**. It is not yet suitable for unattended access to
sensitive personal data or destructive permissions.

See:

- [`docs/architecture.md`](docs/architecture.md)
- [`docs/security-model.md`](docs/security-model.md)
- [`docs/roadmap.md`](docs/roadmap.md)
- [`SECURITY.md`](SECURITY.md)

## License

Kestrel is licensed under the [MIT License](LICENSE).
