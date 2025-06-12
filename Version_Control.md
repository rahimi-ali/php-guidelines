# 🗂️ Version Control

Maintain code quality and team efficiency with these **Git best practices and naming conventions**.

## 🧬 System

- **Version Control:**
  - All projects must use **Git** as the primary (and only) VCS.

## 🌿 Branch Naming Convention

Clear, consistent naming helps everyone quickly understand the purpose of each branch.

| Prefix                      | Use Case                                                | Example                             |
| --------------------------- | ------------------------------------------------------- | ----------------------------------- |
| `feat/<TICKET_ID>/desc`     | New features & improvements                             | `feat/123/user-login-api`           |
| `fix/<TICKET_ID>/desc`      | Bug fixes                                               | `fix/124/fix-login-redirect`        |
| `refactor/<TICKET_ID>/desc` | Internal code changes that do **not** affect behavior   | `refactor/12/remove-obsolete-utils` |
| `chore/<TICKET_ID>/desc`    | Non-production changes: docs, tests, build config, etc. | `chore/13/update-readme`            |

- **Format Rules:**
  - `<TICKET_ID>` is your issue tracker/ticket number (omit for refactor/chore if not tracked). It can be omitted if you don't have a ticket.
  - **Short description** should use **kebab-case** (e.g., `improve-api-response`).
  - Keep it concise and meaningful.
