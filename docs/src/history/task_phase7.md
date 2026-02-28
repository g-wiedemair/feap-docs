# Phase 7: The Engineering Suite

## Phase 7.1: Workbench 3-Pane Layout
- [x] **UI Architecture: Right Sidebar**
    - [x] Add `inspector` panel to `ui/layout.rs`
    - [x] Update `ApplicationState` to track `right_sidebar_width`
    - [x] Implement resize divider for Right Sidebar
- [x] **Left Sidebar Refinement**
    - [x] Wrap Part List in `iced::widget::Scrollable`
    - [x] Ensure vertical filling behavior
- [x] **Content Migration (Split View)**
    - [x] Create `ui/inspector.rs`
    - [x] Move "Results/Legend" logic to Inspector
    - [x] Dedicate Left Sidebar to "Model Structure" (Parts)
- [x] **Refinement**
    - [x] Ensure 3D Viewport resizes correctly between sidebars
    - [x] Apply dark theme consistency

## Phase 7.2: Result Visualization & HUD
- [x] **UI: Inspector Implementation**
    - [x] Add `PickList` for Result selection (None, Displacement, Stress)
    - [x] Add `TextInput` for Legend Min/Max values
    - [x] Add Color Pickers for OOB Min/Max colors
    - [x] Wire inputs to `ApplicationState` updates
- [x] **Core/State Update: Result Mapping**
    - [x] Implement result field mapping to vertex colors in `viewport/mesh.rs`
    - [x] Implement auto-range logic for Min/Max
    - [x] Refactor `update_instances` to use shared color logic
- [x] **WGPU Shader & Pipeline Updates**
    - [x] Update `Vertex` struct to include scalar `data`
    - [x] Update `Shader` to handle 8 discrete color bands
    - [x] Add uniforms for Min/Max/OOB Colors
    - [x] Implement fragment logic for OOB coloring
    - [x] Update `pipeline.rs` uniform structure
- [x] **WGPU: HUD Rendering**
    - [x] Add `glyphon` dependency
    - [x] Implement `viewport/hud.rs` for overlay rendering
    - [x] Render Color Bar (Quad) and Min/Max Text labels

## Phase 7.3: HUD Refinement & Polish
- [x] **HUD Rendering Refinement**
    - [x] Move Legend to Top-Left
    - [x] Render 10 discrete blocks (OOB Min + 8 Steps + OOB Max)
    - [x] Render 10 text labels
- [x] **UI Cleanup**
    - [x] Remove unused `SetOobMinColor` / `SetOobMaxColor` messages
    - [x] Remove `oob_min_color` / `oob_max_color` pickers from Inspector
    - [x] Hardcode/Default OOB colors (Navy/DarkRed)
- [x] **Bug Fix: HUD Positioning**
    - [x] Calculate physical offset (bounds.x * scale) in Primitive::prepare
    - [x] Update HudRenderer::prepare to apply offset and scale factor

## Phase 7.3b: Intelligent Beam Diagram Labels
- [x] **Core/State: Label Collection**
    - [x] Define `DiagramLabel` struct in `diagrams.rs`
    - [x] Update `generate_diagram_mesh` to return peak labels
    - [x] Respect `hidden_parts` in `generate_diagram_mesh`
- [x] **Viewport Logic: Projection & Culling**
    - [x] Create `viewport/labels.rs`
    - [x] Implement `project_to_screen`
    - [x] Implement `filter_visible_labels` (De-Cluttering)
- [x] **Rendering: Text Overlay**
    - [x] Update `HudRenderer::prepare` to accept dynamic labels
    - [x] Integrate label filtering and rendering in `widget.rs`

## Phase 7.5: Visualization Polish & Bug Fixes
- [x] **HUD Polish**
    - [x] Implement Scissor-Rect Clipping for HUD Render Pass
    - [x] Fix stale label rendering when all parts hidden
- [x] **Coordinate Transformations**
    - [x] Fix label projection aspect ratio drift
    - [x] Use physical widget bounds for GL projection mapping
- [x] **State Sync**
    - [x] Update diagram mesh on part visibility toggle
    - [x] Update diagram mesh on View Mode change
- [x] **Documentation**
    - [x] Finalize README, ROADMAP and Implementation History.
- [x] **Project Wide Finalization**
    - [x] Final build verification and cleanup.

## Phase 7.6: Camera, Console & Workflow Refinements
- [x] **Camera Projection Switch**
    - [x] Implement `SetProjection` logic in `app/mod.rs` with FOV/Ortho synchronization
    - [x] Add Perspective/Orthographic toggle to Inspector Display Settings
    - [x] Implement dynamic "Switch View" menu item in `header.rs`
    - [x] Add "Top View (Ortho)" shortcut
- [x] **Console Layout Refinement**
    - [x] Rearrange header: Toggle on left, grouped controls, stats on right
    - [x] Implement full-fidelity rendering for collapsed status bar (Timestamp + Level + Target)
- [x] **Global Project Workflow**
    - [x] Refactor `app/mod.rs` to allow New/Open Project during active editor sessions
    - [x] Coordinate state transitions between Launcher and Editor
