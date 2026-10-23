# Desktop requirements

| ID | Requirement | Priority | Status |
|---|---|---|---|
| DESK-001 | OpenDAP shall have a full-featured desktop app, written in Rust, besides the `opendap` CLI. | must | Confirmed |
| DESK-002 | The desktop app shall let users do everything that can be done with an OpenDAP, from casual use to advanced use. | must | Confirmed |
| DESK-003 | The app shall serve casual users first and keep advanced features available without imposing them. | must | Draft |
| DESK-004 | The app and the `opendap` CLI shall share one core and one domain model (TK-009), so a feature exists once and both expose it. | must | Draft |
| DESK-005 | OpenDAP Desktop shall run on UNIX/POSIX hosts, tested on macOS, like the rest of the project. | must | Confirmed |
| DESK-006 | OpenDAP Desktop shall include a per-user background service on the host, started at login without administrator rights, for work that must happen while the app is closed: device detection, library and playlist sync, backups, update downloads, notifications and persistent connections. | should | Draft |
| DESK-007 | The background service shall be another interface over the same core as the CLI and the app (Rust crate architecture). The app and the CLI use it when it runs; the CLI keeps working without it. | should | Draft |
| DESK-008 | The background service shall integrate with each host's standard service manager and device access (launchd on macOS, systemd user units and udev rules on Linux; Windows equivalents if Windows is supported), and stay light on CPU, memory and battery when idle. | should | Draft |
| DESK-009 | OpenDAP Desktop should also support Windows (split from DESK-005). | should | Confirmed |
| DESK-010 | OpenDAP Desktop should offer optional transcoding per device when syncing, to fit more music, without ever changing the library. | should | Confirmed |
| DESK-011 | Sync and the connected features between the T1 and a computer (over USB, Wi-Fi, Bluetooth or any other link) shall go through OpenDAP Desktop or the `opendap` CLI. Without them, the T1 still offers plain USB storage (T1-065) and complete standalone use (T1-066). | must | Confirmed |

## Context

- DESK-011 (maintainers, 2026-10-23): the client on the computer is what makes sync and connected features possible; the T1 never needs it to play, and plain files over USB storage need no client.
- DESK-010 (maintainers, 2026-10-23), from the [desktop companion research](../research/2026-10-desktop-companion.md) §4.1: whatever is useful may be offered. Off by default; lossless stays lossless unless the listener chooses.
- DESK-006 to DESK-008 (2026-10-22): proposed by Claude after the maintainers asked whether the desktop app needs background services; Draft until confirmed. The MCP server of the CLI spec (`mcp serve`) can run inside this service.
- DESK-003 and DESK-004 (2026-10-22) follow from the request ("from the casual user to the advanced one") and from TK-009 (every toolkit tool shares concepts, conventions and one domain model), applying the brand rule "complexity available, never imposed".
- DESK-005 (maintainers, 2026-10-22): UNIX/POSIX is the main target, tested on macOS (as TK-010, TK-011); Windows is desirable for casual users, so the app avoids POSIX-only assumptions in its core and keeps Windows support possible.
- Scope to define: music library and sync, device setup and updates, settings, faceplates and HATs, capabilities, backups and recovery, and the developer features the CLI covers (shell, logs, flashing, virtual T1).
