# Contributing to Kestrel

Kestrel is in an early design and prototyping stage. Contributions that preserve its local-first,
least-privilege, human-controlled design are welcome.

## Before contributing

- Do not commit real credentials, private hostnames, personal data, or private document contents.
- Prefer configuration through environment variables or clearly documented local overrides.
- Avoid introducing destructive permissions when a read-only or approval-gated alternative is
  practical.
- Keep integrations modular so users are not forced into a particular cloud provider or personal
  information system.

## Development workflow

Until a more formal workflow is established:

1. Create a focused branch.
2. Keep commits small and descriptive.
3. Run the repository's secret checks before pushing.
4. Open a pull request describing behavior changes and security implications.
