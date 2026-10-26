# Roadmap

Status legend: ⬜ not started · 🟨 in progress · ✅ done

Planned pace: ~1 h on weekdays, more on weekends. Milestones are split into
small sub-milestones that fit a few sessions each.

## Milestone 1 — Core domain model (`clip-core`) 🟨

Rust focus: Cargo workspaces, modules, structs/enums, ownership and borrowing,
`String` vs `&str`, `Option`/`Result`, custom error types, `VecDeque`,
iterators, unit tests.

### 1.1 — Workspace and basic history ⬜
- Cargo workspace with library crate `crates/clip-core`.
- `ClipEntry { id, content, created_at, pinned }`.
- `History::new(capacity)`, `push`, `len`, `is_empty`, `iter` (newest first).
- Unique, increasing ids that are never reused.
- Empty or whitespace-only content is ignored.

**Acceptance:** tests for empty history, single push, ordering, id
uniqueness, empty-content rejection; fmt and clippy clean.

### 1.2 — Rules: dedup, pin, eviction, errors, search ⬜
- Pushing existing content moves it to the front (no duplicate).
- Capacity overflow evicts the oldest **unpinned** entry; pinned entries are
  never evicted (behavior when all entries are pinned is decided and
  documented by the developer).
- `pin`/`unpin`/`remove`/`get` by id; errors via `HistoryError` enum
  implementing `Display` and `std::error::Error`.
- `search(query)`: case-insensitive substring match, returns an iterator of
  references (no cloning).

**Acceptance:** all tests from the Milestone 1.2 brief pass; no `unsafe`, no
external dependencies, no cloning in `iter`/`search`; fmt and clippy clean.

## Milestone 2 — Persistence (`clip-store`) ⬜
Rust focus: traits, error conversion with `From` and `?`, external crates,
integration tests, temp files.
- `HistoryStore` trait; SQLite implementation (`rusqlite`); schema migrations.
- Load on startup, save on change; file under `$XDG_DATA_HOME` with mode 0600.

## Milestone 3 — Clipboard capture spike and backend (`clip-backend`) ⬜
Rust focus: trait objects vs generics, threads, channels (`std::sync::mpsc`),
FFI-backed crates, feature flags.
- Research spike: how to observe the clipboard on GNOME Wayland (XWayland +
  XFixes, data-control, or GNOME Shell extension). Written conclusion in
  `ARCHITECTURE.md`.
- `ClipboardSource` / `ClipboardSink` traits, one real backend, one mock.

## Milestone 4 — Daemon (`clip-daemon`) ⬜
Rust focus: concurrency (`Arc`, `Mutex`, message passing), async basics,
signal handling, logging (`tracing`), IPC (D-Bus via `zbus` or Unix socket).
- Event loop wiring backend → history → store.
- IPC API: list, search, pin, remove, select.
- Graceful shutdown.

## Milestone 5 — UI (`clip-ui`, GTK4 + libadwaita) ⬜
Rust focus: GTK object model, closures and `'static`, `Rc`/`RefCell`,
main-loop integration.
- Popup window with list, search field, keyboard navigation, pin toggle.
- Selecting an entry sets the clipboard and closes the window.

## Milestone 6 — Desktop integration ⬜
- GNOME custom shortcut setup, `.desktop` file, systemd user service /
  autostart.
- Privacy: password-manager hint, excluded apps, pause, clear history.
- Config file (TOML, `$XDG_CONFIG_HOME`).

## Milestone 7 — Rich content ⬜
- Images (`image/png`) with thumbnails and size limits; HTML as text fallback.

## Milestone 8 — Hardening and release ⬜
- CI (GitHub Actions: fmt, clippy, test), packaging, performance review,
  optional GlobalShortcuts portal and auto-paste research.
