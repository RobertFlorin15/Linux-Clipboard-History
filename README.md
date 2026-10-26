# Linux Clipboard History

A clipboard history manager for Linux, inspired by Windows Clipboard History
(Win+V), written in Rust.

> **Status:** early development — Milestone 1 (core domain model).
> This is also a learning project; see `LEARNING.md`.

## Planned features

- Persistent clipboard history (text first, images later)
- Popup window with search and full keyboard navigation
- Pinned entries that are never evicted
- Privacy controls: ignore password managers, excluded apps, pause, clear
- Native GNOME look (GTK4 + libadwaita); Wayland-first, X11 supported

## Requirements

- Rust stable (edition 2024) with `rustfmt` and `clippy`:
  <https://rustup.rs>
- Later milestones: GTK4 and libadwaita development packages
  (`libgtk-4-dev`, `libadwaita-1-dev` on Ubuntu/Debian)

## Development

```sh
cargo build
cargo test
cargo fmt --check
cargo clippy --all-targets -- -D warnings
```

## Documentation

- `ARCHITECTURE.md` — components, decisions, Linux-specific risks
- `ROADMAP.md` — milestones and acceptance criteria
- `LEARNING.md` — concepts studied, assignments, progress
- `AGENTS.md` — mentoring and development rules for AI assistants

## License

To be decided.
