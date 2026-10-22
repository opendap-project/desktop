# Desktop requirements

| ID | Requirement | Priority | Status |
|---|---|---|---|
| DESK-001 | OpenDAP shall have a full-featured desktop app, written in Rust, besides the `opendap` CLI. | must | Confirmed |
| DESK-002 | The desktop app shall let users do everything that can be done with an OpenDAP, from casual use to advanced use. | must | Confirmed |
| DESK-003 | The app shall serve casual users first and keep advanced features available without imposing them. | must | Draft |
| DESK-004 | The app and the `opendap` CLI shall share one core and one domain model (TK-009), so a feature exists once and both expose it. | must | Draft |

## Context

- DESK-003 and DESK-004 (2026-10-22) follow from the request ("from the casual user to the advanced one") and from TK-009 (every toolkit tool shares concepts, conventions and one domain model), applying the brand rule "complexity available, never imposed".
- Supported hosts: development officially supports UNIX/POSIX hosts tested on macOS (TK-010, TK-011). Whether the desktop app, which serves casual users, also targets Windows is an open question for the maintainers.
- Scope to define: music library and sync, device setup and updates, settings, faceplates and HATs, capabilities, backups and recovery, and the developer features the CLI covers (shell, logs, flashing, virtual T1).
