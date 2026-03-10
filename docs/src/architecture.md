# Architecture

FEAP follows a **stratified architecture**, allowing for a clear separation of concerns between engineering logic, scripting, and the user interface [1]. Recent updates have shifted the core data structures to a **Struct of Arrays (SoA)** layout, drastically improving memory efficiency and cache coherence for large-scale  models.

## Crate Structure

### `feap_core` (Core Domain Layer)
A pure Rust library containing all FEA physics, material models, and numerical solvers [1].
* **Model**: Central data structure managing nodes, elements, materials, sections, loads, and results [1]. It utilizes a **Struct of Arrays (SoA)** design (HashMaps for coordinates, fixity, loads) to ensure extremely fast iteration during matrix assembly.
* **Material Architecture**: Efficiently manages constitutive laws via a unified `MaterialType` enum and borrowed references, eliminating redundant heap allocations (`Box<dyn Material>`) per element. This architecture computes local stress-strain responses and handles complex state history for non-linear return-mapping (e.g., Elasto-Plasticity and upcoming geotechnical models like Mohr-Coulomb).
* **Element Trait**: Encapsulates physical behaviors (stiffness, force vectors, load vectors) for element types like 3D `Truss`, `Beam`, and the newly added 2D continua elements (`Quad4`, `Quad8`) [1].
* **Solver Trait**: Unified interface for solvers [1]. The `NewtonRaphsonSolver` implements nonlinear iteration and delegates the tangent system to a pluggable `LinearSystemSolver` backend (Pardiso, Skyline, Frontal, Dense, SuiteSparse) [1]. 

### `feap_gui` (Visualization Layer)
Utilizes the **Iced framework** and **wgpu** to deliver high-performance 3D rendering at 60 FPS across Vulkan, Metal, and DirectX backends [2].
* **ApplicationState**: Manages all UI state (camera, selection, view modes, load case index) [2].
* **Viewport**: Custom `wgpu` shader pipeline for mesh, loads, diagrams, and deformed shapes [2], now featuring accurate nodal stress recovery for 2D continua.
* **Binary Caching**: Internal result caching has been upgraded from JSON/MsgPack to high-performance **Bincode**. This drastically reduces RAM consumption and allows for near-instant switching between complex load cases.

### `feap_scripting` (Scripting Layer)
Uses **PyO3 bindings** to expose the Rust core for interactive scripting and automation [3].
* **PyModel**: Python wrapper around `feap_core::Model` providing methods for node/element creation, load application, solving, and result extraction [3].
* **Embedded Engine**: Python runs in-process — no serialization overhead [3], establishing a seamless Foreign Function Interface (FFI) bridge.
* **Import-Only Mode**: An intelligent "Dry-Run" logic allows users to load complex Python scripts (even those containing `model.solve()`) into the GUI instantly, bypassing the long batch execution for quick geometry inspection.

## Solver Architecture

The `NewtonRaphsonSolver` implements the `Solver` trait and manages [3]:
1. **Equation assignment**: Fast DOF numbering and masking for 2D/3D system types [3].
2. **Load stepping**: Incrementally applying absolute target loads using smooth interpolation to avoid numerical shocks.
3. **State Inheritance**: Injecting the full physical equilibrium (displacements, total strains, stresses, and plastic history) from previous construction stages into the current model setup.
4. **Iterative convergence**: Checking force and displacement tolerances [3].
5. **Result storage**: Saving converged states into independent result buffers per load case [3].
6. **Reaction force computation**: Evaluating $R = Ku - F_{ext}$ [3].

## Data Flow

1. **Input**: TOML file or Python Script -> `feap_core` parser -> `Model` struct [4].
2. **Cache**: The parsed structure is immediately cached using **Bincode** (`.bin`), enabling zero-overhead reloading for large projects.
3. **Solve**: `NewtonRaphsonSolver::solve(&mut model)` calculates equilibrium and populates `model.results` [4].
4. **Display**: `feap_gui` reads `model.results` to render deformed shapes, force diagrams, and color maps [4].
