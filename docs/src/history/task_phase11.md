# Phase 11: Python API Refinement & Advanced Elasto-Plasticity ✅

## Python API Extension & Refinement ✅
- [x] **PyO3 0.23 Migration**: Updated the entire `feap_scripting` crate to the latest PyO3 version, ensuring compatibility with Python 3.13 and modern memory management (`Bound` API).
- [x] **Universal Displacement Extraction**: Implemented `get_node_displacement` for targeted nodal result harvesting.
- [x] **Internal Force Extraction**: Added `get_element_forces` to query axial, shear, and moments at arbitrary relative positions [0..1] along any element.
- [x] **Reaction Engine Extraction**: Implemented `get_support_reaction` to extract 6-DOF reaction vectors from constrained nodes.
- [x] **Result Set Abstraction**: Refactored internal result handling in Python to allow querying specific `load_case` IDs or falling back to the active GUI case.

## True Nonlinear Stress Recovery ✅
- [x] **Fiber-Level Integration**: Implemented a new `evaluate_fiber_stress` trait method to compute the exact physical stress state at any $(y, z)$ coordinate within a cross-section.
- [x] **Elasto-Plastic Force Recovery**: Updated element integration to calculate internal forces by numerically integrating true fiber stresses when in the plastic regime, rather than relying on linear Navier fallbacks.
- [x] **Heatmap Accuracy**: Refined GUI heatmap sampling (8 points per slice for circles/pipes) to ensure yielding peaks are captured and visualized realistically (capped by $f_y$).

## Solver Robustness & Persistence ✅
- [x] **Plastic Strain Persistence**: Fixed a critical bug where converged Newton-Raphson states were not committed to the history, causing loss of plastic strain data.
- [x] **Regression Tests**: Added `test_fiber_beam_yielding` to the core suite to verify history variable transition and prevent future regressions of plastic strain tracking.
- [x] **TOML Consistency**: Resolved convergence parity issues between raw Python-generated models and restored TOML projects.

## GUI Visual & Workflow Polish ✅
- [x] **Right-Handed Rotation Matrix**: Corrected a left-handed determinant bug in the instance rendering pipeline, ensuring beam orientations are visually correct.
- [x] **Window Persistence 2.0**: Implemented asynchronous `Task`-based window maximization on startup and guarded coordinate saving to prevent OS-level window movement from corrupting restored positions.
