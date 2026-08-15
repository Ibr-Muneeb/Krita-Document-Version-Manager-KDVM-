# Krita Checkpoints

A lightweight, checkpoint-based version history plugin for [Krita](https://krita.org/), built as a native Krita dockable widget in PyQt5.

Krita has no built-in way to step back through a document's history the way `git log` lets you step through a codebase. This plugin adds that: one click saves a timestamped, thumbnailed snapshot of the current `.kra` file; the history panel lets you browse, restore, rename, or delete any snapshot without leaving Krita.


![demo](docs/demo.gif)

## Why this exists

Digital painters iterate a lot — Krita's undo stack doesn't survive closing the file, and manually duplicating `.kra` files as `painting_v2_final_FINAL.kra` doesn't scale. This plugin gives Krita documents a minimal, local, per-file version history: no external tools, no git, no cloud service.

## Features

- **One-click checkpoints** — snapshot the current document with an optional message; auto-generates a thumbnail.
- **Animation-aware thumbnails** — for documents with an animation timeline, the thumbnail is a filmstrip of evenly-sampled frames instead of a single flattened image, so animated checkpoints are recognizable at a glance (🎞 badge in the history panel).
- **History panel** — a sortable table of every checkpoint for the active document, newest first, with adjustable thumbnail size.
- **Restore workflows** — *Load Checkpoint* opens a snapshot alongside your current work; *Make Current* replaces the active document with a chosen checkpoint (and checkpoints the restore itself, so nothing is lost).
- **In-place editing** — rename a checkpoint's message after the fact, or regenerate its thumbnail.
- **Housekeeping** — delete old checkpoints, or jump straight to a checkpoint's folder in your OS file browser.
- **Reactive to Krita** — the panel automatically reloads when you switch documents or Krita autosaves a file.

## Architecture

The plugin is a small MVC-ish split across four modules:

```
krita_checkpoints/
├── __init__.py            # registers the dock widget with Krita's plugin system
├── qt_docker_widget.py     # QtDocker (Krita DockWidget) + VersionManager (top-level panel UI/logic)
├── qt_history_widget.py    # HistoryModel (QAbstractTableModel) + HistoryWidget (table + context menu)
├── utils.py                 # Utils — all filesystem/JSON I/O, locking, and checkpoint bookkeeping
├── qt_docker_widget_ui.py   # Qt Designer-generated layout (do not hand-edit; regenerate from the .ui)
└── icons_rc.py               # compiled Qt resource file (icons)
```

**Data model:** each managed document gets a sibling directory, `<document>.kra.d/`, containing a `history.json` index and one subdirectory per checkpoint (a copy of the `.kra` file at that point in time, plus its thumbnail). `QLockFile` guards `history.json` against concurrent writes.

**Signal flow:** `Utils` emits Qt signals for status/error events rather than touching the UI directly, so the filesystem layer stays independent of the widget layer — `HistoryWidget` and `VersionManager` just relay those signals up to the debug console / message boxes.

## Installation

1. Copy `krita_checkpoints/` and `krita_checkpoints.desktop` into your Krita `pykrita` resources folder:
   - Linux: `~/.local/share/krita/pykrita/`
   - Windows: `%APPDATA%\krita\pykrita\`
   - macOS: `~/Library/Application Support/krita/pykrita/`
2. In Krita: **Settings → Configure Krita → Python Plugin Manager**, enable **Krita Checkpoints**, restart Krita.
3. Enable the dock: **Settings → Dockers → Document Version Manager**.

## Usage

1. Save your `.kra` document (checkpoints require a filename).
2. Type an optional message, click **add checkpoint**.
3. Browse checkpoints in the panel; right-click a row for restore/rename/delete options.

## Requirements

- Krita 5.x (bundles Python 3 + PyQt5 — no separate install needed)
- No external Python dependencies for the current feature set

## Roadmap

- [ ] Unit tests around `Utils` (pure filesystem/JSON logic, testable without a running Krita instance).
- [ ] GitHub Actions workflow: `black`/`flake8` on push.
- [ ] Optional checkpoint tagging/branching for divergent exploration.

## License

GPL-3.0-or-later — see [LICENSE](LICENSE).

---
*Originally forked and restructured from an earlier prototype; cleaned up for public release (normalized line endings, removed dead code, fixed two UI state bugs in the log-view toggle and the delete-checkpoint cancel path).*
