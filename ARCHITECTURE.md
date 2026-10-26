# Architecture

## Overview

A background **daemon** watches the system clipboard and records history. A
short-lived **UI** popup (Win+V style) shows the history, supports search,
keyboard navigation, and pinning, and puts the chosen entry back on the
clipboard.

```
             ┌──────────────────────────── clip-daemon ───────────────────────────┐
 system      │                                                                     │
 clipboard ──┼─► clip-backend ──(events)──► History (clip-core) ◄──► clip-store   │
 (Wayland/   │   ClipboardSource                     │                (SQLite)   │
  X11)    ◄──┼── ClipboardSink                       │                           │
             │                                  IPC server                       │
             └───────────────────────────────────────┼───────────────────────────┘
                                                     │ D-Bus / Unix socket
                                                clip-ui (GTK4 + libadwaita)
```

## Workspace layout (planned)

| Crate          | Kind | Responsibility                                                    | Depends on            |
|----------------|------|-------------------------------------------------------------------|-----------------------|
| `clip-core`    | lib  | Domain model: `ClipEntry`, `History`, dedup, pin, eviction, search. **No I/O.** | — |
| `clip-store`   | lib  | Persistence behind a `HistoryStore` trait; SQLite implementation. | `clip-core`           |
| `clip-backend` | lib  | `ClipboardSource` / `ClipboardSink` traits + platform backends + mock. | `clip-core`      |
| `clip-daemon`  | bin  | Event loop, owns `History`, IPC server, privacy filters.          | core, store, backend  |
| `clip-ui`      | bin  | Popup window: list, search, keyboard navigation, pin, select.     | `clip-core`, IPC      |

Crates are created incrementally, one per milestone (see `ROADMAP.md`).

## Design principles

- **Pure core.** All business rules live in `clip-core` and are unit-testable
  without a display server, database, or async runtime.
- **Traits at system boundaries.** Storage and clipboard access are behind
  traits so backends can be swapped (Wayland / X11 / GNOME) and mocked in tests.
- **Daemon owns state.** On X11 clipboard data lives in the source application
  and disappears when it exits; the daemon eagerly copies content on change.
- **Privacy by default.** Database file mode `0600` under `$XDG_DATA_HOME`;
  respect password-manager hints; support excluded apps and pause.
- **Low resource usage.** Event-driven (no polling where avoidable); bounded
  history size and bounded entry size.

## Architectural decisions

### ADR-001: Cargo workspace with multiple crates
- **Decision:** one workspace, separate crates per responsibility.
- **Why:** enforces dependency direction at compile time, keeps the core pure,
  and makes each milestone a self-contained learning unit.

### ADR-002: Separate daemon and UI processes
- **Why:** history capture must run continuously; the UI is shown on demand.
  A crash in the UI cannot lose history.

### ADR-003: GUI toolkit — GTK4 + libadwaita (`gtk4-rs`, `libadwaita-rs`)
- **Why:** native look and behavior on GNOME (primary target), mature
  accessibility and input-method support, Wayland-first.
- **Trade-off:** GObject subclassing is less idiomatic than pure-Rust toolkits.
- **Alternative considered:** iced (pure Rust, Elm architecture).

### ADR-004: Hotkey and paste strategy for v1
- **Hotkey:** a GNOME custom keyboard shortcut runs the UI. Later: the
  `org.freedesktop.portal.GlobalShortcuts` portal.
- **Paste:** selecting an entry sets the clipboard; the user presses Ctrl+V.
  Wayland forbids synthetic key injection into other apps; auto-paste may come
  later via the RemoteDesktop portal or `uinput`.

### ADR-005: Persistence — SQLite (planned, Milestone 2)
- **Why:** transactional, single file, efficient search, mature Rust bindings
  (`rusqlite`). To be confirmed at Milestone 2.

## Linux-specific risks

| Risk | Notes | Mitigation |
|------|-------|------------|
| GNOME (Mutter) has historically not exposed `ext-data-control-v1` / `wlr-data-control` | Unfocused Wayland clients cannot read the clipboard without it | Research spike (Milestone 3): XWayland + XFixes selection events, a GNOME Shell extension bridge, or data-control if available in current GNOME |
| No global hotkeys on Wayland | Clients cannot grab keys globally | GNOME custom shortcut; later GlobalShortcuts portal |
| No synthetic input on Wayland | Cannot auto-paste | Manual Ctrl+V in v1 |
| No window positioning on Wayland | Cannot place window at cursor | Centered window |
| X11 selection ownership | Data vanishes when source app exits | Daemon copies content eagerly |
| Multiple MIME types per copy | `text/plain`, `text/html`, `image/png`, … | Preferred-format selection, size limits |
| Sensitive data | Passwords copied from password managers | `x-kde-passwordManagerHint`, exclusions, pause, file permissions |
| PRIMARY vs CLIPBOARD | Mouse selection vs Ctrl+C | Track CLIPBOARD only by default |
