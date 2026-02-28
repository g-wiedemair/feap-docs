# Phase 6 Walkthrough: Core Refactoring & Project Workflow

## Overview

Phase 6 introduced two major threads of work: a structural refactoring of the core data model and a complete project management workflow for the GUI.

## Core Refactoring

### Data-Oriented Design
The `Model` struct was refactored from an object-oriented layout (with `Node` structs holding fixity, loads, etc.) to a **Struct of Arrays (SoA)** design. This means coordinates, fixity conditions, loads, and displacements are now stored as separate `HashMap`s keyed by node ID, enabling more efficient iteration and simpler serialization.

### TOML Input Simplification
The input format was simplified to use flat TOML tables:
```toml
[nodes]
1 = [0.0, 0.0, 0.0]
2 = [10.0, 0.0, 0.0]

[fixity]
1 = [true, true, true]

[loads]
2 = [0.0, -1000.0, 0.0]
```

### Solver Adaptation
`integrate_stiffness`, `assemble_global_stiffness`, and `NewtonRaphsonSolver` were updated to work with the new SoA `Model` layout.

## GUI: Project Management

### Launcher & Editor States
`FeapApp` was restructured to support two application states: **Launcher** (project selection) and **Editor** (active workspace). The launcher displays recent projects with thumbnails and provides New/Open actions.

### Project Persistence
- `AppConfig` saves recent project paths to `recent_projects.json`
- Projects are saved/loaded as `.toml` files with full round-trip fidelity

### Menu Bar
A menu bar was implemented with File operations (New, Open, Save, SaveAs, Close) and keyboard shortcuts (Ctrl+N, Ctrl+O, Ctrl+S, Ctrl+Shift+S).

### WGPU Screenshot Thumbnails
Project thumbnails are generated via WGPU offscreen rendering:
1. On save, `screenshot_path` is set to `{project_name}_thumb.png`
2. The next `prepare()` call renders the scene to an offscreen texture
3. `capture_texture()` copies the GPU texture to a CPU buffer and saves as PNG
4. The launcher displays these thumbnails in the recent projects list

## Files Changed (Key)

| File | Change |
|------|--------|
| `feap_core/src/model.rs` | SoA refactoring |
| `feap_core/src/input.rs` | TOML format update |
| `feap_gui/src/app/mod.rs` | Launcher/Editor state machine |
| `feap_gui/src/state/mod.rs` | ApplicationState with screenshot_path |
| `feap_gui/src/viewport/screenshot.rs` | **NEW** — WGPU texture capture |
| `feap_gui/src/viewport/widget.rs` | Offscreen rendering for thumbnails |
| `feap_gui/src/ui/launcher.rs` | Thumbnail display in recent projects |
| `feap_gui/src/ui/menu.rs` | Menu bar implementation |

## Verification
- `cargo check --lib` passes with 0 errors, 0 warnings
- `cargo test` passes for core refactoring changes
- Thumbnails generated and displayed correctly on save
