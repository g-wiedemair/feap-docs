# Phase 6: Core Refactoring & Project Workflow

This phase focused on refactoring the core data model to a data-oriented design, modernizing the input format, and completing the GUI project management workflow including thumbnails.

## Completed Tasks

### 1. Core Refactoring: Data-Oriented Design
- [x] Slim down `Node` struct to just ID/Coords (transient)
- [x] Refactor `Model` to pure Struct of Arrays (SoA)
    - Coords, Fixity, Loads, Displacements as HashMaps
- [x] Update `Element` trait to work with new Model layout

### 2. Input Format Redesign (TOML)
- [x] Update `InputModel` to parse simplified `[nodes]`, `[fixity]`, `[loads]` tables
- [x] Update `load_model` normalization logic

### 3. Solver & Assembly Adaptation
- [x] Refactor `integrate_stiffness` to use Model data
- [x] Update `assemble_global_stiffness` and `NewtonRaphson`

### 4. GUI Upgrade: Project Management
- [x] UX Redesign: Launcher (Refactor `FeapApp` state, Launcher vs Editor)
- [x] Update `iced` styling to 0.13 API
- [x] Fix Layout & Console Logging
- [x] Implement Menu Bar with keyboard shortcuts
- [x] `Config` persistence (`recent_projects.json`)
- [x] WGPU Screenshot-based project thumbnails
