# Architecture

FEAP follows a **stratified architecture**, allowing for a clear separation of concerns between engineering logic, scripting, and the user interface [1, 2, 5].

## Crate Structure

```
feap/
├── feap_core       # Pure Rust FEA engine (no GUI dependencies)
├── feap_gui        # Interactive 3D editor (Iced + wgpu)
├── feap_scripting  # Python bridge via PyO3
└── docs/           # mdBook documentation
```

### feap_core (Core Domain Layer)
A pure Rust library containing all FEA physics, material models, and numerical solvers [1, 2].

- **Model**: Central data structure managing nodes, elements, materials, sections, loads, and results.
- **Element Trait**: Encapsulates physical behaviors (stiffness, force vectors, load vectors) for element types like `Truss` and `Beam`.
- **Solver Trait**: Unified interface for solvers. The `NewtonRaphsonSolver` implements nonlinear iteration and delegates the tangent system to a pluggable `LinearSystemSolver` backend (Pardiso, Skyline, Frontal, Dense, SuiteSparse).
- **Input/Output**: TOML parsing, binary MsgPack caching, and `.out` result file generation.

### feap_gui (Visualization Layer)
Utilizes the **Iced framework** and **wgpu** to deliver high-performance 3D rendering at 60 FPS across Vulkan, Metal, and DirectX backends [5, 6].

- **ApplicationState**: Manages all UI state (camera, selection, view modes, load case index).
- **Viewport**: Custom `wgpu` shader pipeline for mesh, loads, diagrams, and deformed shapes.
- **Inspector**: Right sidebar with results visualization and object inspection.
- **Header**: Load case selector with integrated ▶ Run / ▶▶ All solver buttons.

### feap_scripting (Scripting Layer)
Uses **PyO3 bindings** to expose the Rust core for interactive scripting and automation [3, 7].

- **PyModel**: Python wrapper around `feap_core::Model` providing methods for node/element creation, load application, solving, and result extraction.
- **Embedded Engine**: Python runs in-process — no serialization overhead.

## Solver Architecture

```
                    ┌──────────────────────┐
                    │  NewtonRaphsonSolver  │
                    │  (Nonlinear Outer)    │
                    └─────────┬────────────┘
                              │ delegates tangent solve to
              ┌───────────────┼───────────────┐
              │               │               │
        ┌─────┴─────┐  ┌─────┴─────┐  ┌──────┴──────┐
        │  Pardiso   │  │  Skyline  │  │   Frontal   │
        │  (MKL FFI) │  │  (Rust)   │  │   (Rust)    │
        └───────────┘  └───────────┘  └─────────────┘
```

The `NewtonRaphsonSolver` implements the `Solver` trait and manages:
1. Equation assignment (DOF numbering)
2. Load stepping with configurable increments
3. Iterative convergence (force and displacement tolerances)
4. Result storage (displacements per load case)
5. Reaction force computation (R = Ku − F_ext)

## Data Flow

1. **Input**: TOML file → `feap_core::input` parser → `Model` struct
2. **Cache**: First parse creates a binary `.bin` cache (MsgPack) for instant reload
3. **Solve**: `NewtonRaphsonSolver::solve(&mut model)` populates `model.results`
4. **Output**: `feap_core::output` writes `.out` result files
5. **Display**: `feap_gui` reads `model.results` to render deformed shapes, force diagrams, and color maps
