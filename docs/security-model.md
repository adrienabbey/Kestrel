# Security Model

## Core principle

Kestrel should be able to **understand broadly but act narrowly**.

LLM instructions are not a security boundary. Permissions must be enforced by software outside the
model.

## Initial permission model

| Capability | Initial policy |
| --- | --- |
| Read approved tasks | Allow |
| Read approved calendars | Allow |
| Read approved notes/documents | Allow when integrations are added |
| Search approved personal data | Allow |
| Recommend actions | Allow |
| Propose new tasks | Allow |
| Create new tasks | Human approval required |
| Modify existing tasks | Not exposed |
| Complete existing tasks | Not exposed |
| Delete tasks | Not exposed |
| Modify calendars/email/notes/files | Not exposed initially |

## Write controls

When task creation is introduced, it should pass through a policy gateway that can enforce controls
such as:

- explicit human approval;
- a configurable maximum number of tasks per approval;
- configurable rate limits;
- schema and field validation;
- duplicate detection;
- source/provenance recording;
- audit logging.

The write connector should possess only the minimum server-side permissions needed for the permitted
operation whenever the backing service supports that separation.

## Trust boundaries

Kestrel assumes that model output may be incorrect, malformed, manipulated by retrieved content, or
unexpectedly broad. Retrieved email, documents, notes, and webpages must be treated as untrusted
input rather than instructions.

Destructive tools should not merely be hidden from the user interface; they should be absent from
the model-accessible capability surface unless explicitly introduced later with a documented threat
model.

## Secrets

- Keep secrets outside Git.
- Prefer dedicated application credentials over primary account passwords.
- Use least-privilege credentials and revoke them when no longer needed.
- Do not bake internal network names, private addresses, or personal deployment details into public
  configuration when variables or local overrides are sufficient.
