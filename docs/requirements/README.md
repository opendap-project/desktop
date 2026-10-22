# Desktop requirements

| ID | Requirement | Priority | Status |
|---|---|---|---|
| DESK-001 | OpenDAP shall have a full-featured desktop app, written in Rust, besides the `opendap` CLI. | must | Confirmed |
| DESK-002 | The desktop app shall let users do everything that can be done with an OpenDAP, from casual use to advanced use. | must | Confirmed |
| DESK-003 | The app shall serve casual users first and keep advanced features available without imposing them. | must | Draft |
| DESK-004 | The app and the `opendap` CLI shall share one core and one domain model (TK-009), so a feature exists once and both expose it. | must | Draft |
| DESK-005 | OpenDAP Desktop shall run first on UNIX/POSIX hosts, tested on macOS, like the rest of the project, and should also support Windows. | must | Confirmed |
| DESK-006 | OpenDAP Desktop shall include a per-user background service on the host, started at login without administrator rights, for work that must happen while the app is closed: device detection, library and playlist sync, backups, update downloads, notifications and persistent connections. | must | Draft |
| DESK-007 | The background service shall be another interface over the same core as the CLI and the app (Rust crate architecture). The app and the CLI use it when it runs; the CLI keeps working without it. | must | Draft |
| DESK-008 | The background service shall integrate with each host's standard service manager and device access (launchd on macOS, systemd user units and udev rules on Linux; Windows equivalents if Windows is supported), and stay light on CPU, memory and battery when idle. | must | Draft |

## Context

- DESK-006 to DESK-008 (2026-10-22): proposed by Claude after the maintainers asked whether the desktop app needs background services; Draft until confirmed. The MCP server of the CLI spec (`mcp serve`) can run inside this service.
- DESK-003 and DESK-004 (2026-10-22) follow from the request ("from the casual user to the advanced one") and from TK-009 (every toolkit tool shares concepts, conventions and one domain model), applying the brand rule "complexity available, never imposed".
- DESK-005 (maintainers, 2026-10-22): UNIX/POSIX is the main target, tested on macOS (as TK-010, TK-011); Windows is desirable for casual users, so the app avoids POSIX-only assumptions in its core and keeps Windows support possible.
- Scope to define: music library and sync, device setup and updates, settings, faceplates and HATs, capabilities, backups and recovery, and the developer features the CLI covers (shell, logs, flashing, virtual T1).
