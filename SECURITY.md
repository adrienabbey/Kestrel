# Security Policy

Kestrel is designed to interact with sensitive personal information, so security boundaries are part
of the core architecture rather than an optional feature.

## Current status

Kestrel is early experimental software. Until the project documents otherwise:

- deploy it only in environments you trust;
- do not expose development services directly to the public Internet;
- do not grant an LLM destructive permissions to source systems;
- use dedicated, least-privilege credentials for integrations;
- do not store production secrets in the repository.

## Reporting a vulnerability

Please **do not publish credentials, personal data, exploit details, or other sensitive information
in a public GitHub issue**.

When GitHub Private Vulnerability Reporting is enabled for this repository, use that mechanism for
security reports. Until then, open a minimal issue stating that you need a private security contact,
without including vulnerability details.

## Secret exposure

If a real credential is ever committed to Git history, treat that credential as compromised. Rotate
or revoke it immediately; removing it in a later commit does not make the original secret safe.
