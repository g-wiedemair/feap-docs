# Walkthrough - Phase 3: Advanced Solver & Pardiso Integration

Phase 3 transitions FENDA-Rust from a simple linear tool to a high-performance non-linear finite element platform. This walkthrough explains the key technical changes.

## 1. Modular Solver Architecture
The numerical engine now uses a modular design in `feap_core/src/solver/`:
- **`mod.rs`**: Defines the `Solver` trait, allowing the application to swap between linear, NR, and Pardiso solvers seamlessly.
- **`linear.rs`**: Legacy dense solver for quick verification.
- **`nonlinear.rs`**: The new Newton-Raphson numerical core.

## 2. Sparse Matrix Assembly (`sprs`)
To handle large-scale models, we've implemented sparse matrix assembly:
- **`sparse.rs`**: Contains the `SparseAssembler`. It maps nodal DOFs to global equation numbers and assembles contributions into a CSR (Compressed Sparse Row) matrix.
- This significantly reduces memory usage and enables the use of efficient sparse solvers.

## 3. Intel Pardiso direct Solver
We've integrated the industry-standard **Intel Pardiso** direct solver:
- **`pardiso.rs`**: Provides FFI bindings to MKL 2025.3.
- It handles the three distinct phases of direct solving: Analysis, Numerical Factorization, and Solve.
- **Usage**: Compile with `cargo build --features pardiso`.

## 4. Parallel Assembly (Rayon)
Element-level computations (numerical integration and stiffness matrix generation) are now parallelized:
- Using **Rayon**, we distribute element stiffness calculations across all available CPU cores.
- This results in near-linear speedup during the assembly phase of the solver.

## 5. Non-linear Capability (Newton-Raphson & Total Lagrange)
The solver now supports full geometric non-linearity:
- **Newton-Raphson**: Iterative loop with energy and force residual checking.
- **Total Lagrange**: The Truss element has been upgraded to a Total Lagrange formulation, allowing it to handle large rotations and large displacements.
- **Adaptive Arc-Length**: Implemented for capturing snap-through and snap-back behavior efficiently.

## Verification benchmarks
Verification was performed using specialized benchmarks in `feap_core/src/tests.rs`:
- **Linear Test**: Ensures NR converges exactly in one step for linear cases.
- **Snap-Through Test**: A 2-degree-of-freedom shallow arch truss problem was used to verify the Arc-Length solver's ability to navigate limit points. The results correlate with analytical solutions.

---
**Next Milestone**: Phase 4 - Python Integration (PyO3) and Interactive Scripting.
