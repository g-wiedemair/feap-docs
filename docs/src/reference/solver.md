# Solver Reference

FEAP provides a two-level solver architecture: a **nonlinear outer solver** (Newton-Raphson) that manages load stepping and convergence, and pluggable **linear backends** for the tangent system solutions [26, 27].

## Newton-Raphson Solver (Nonlinear)

The Newton-Raphson solver is the primary solver used by both the GUI and Python scripting. It handles geometric and material nonlinearities through iterative load stepping.

### Algorithm
1. **Equation assignment**: Map unconstrained DOFs to equation numbers.
2. **Load stepping**: Incrementally apply loads over configurable steps.
3. **Iteration**: For each step, assemble the tangent stiffness, compute the residual, solve for displacement increments, and check convergence.
4. **Post-processing**: Compute reaction forces at constrained nodes.

### Convergence Criteria
- **Force tolerance**: Residual norm < 1e-8 (default).
- **Displacement tolerance**: Increment norm < 1e-10 (default).
- **Max iterations**: 50 (default).

### Usage
The Newton-Raphson solver wraps any of the linear backends listed below. The backend is selected via the `[config]` section:

```toml
[config]
solver = "Pardiso"  # Linear backend: "Pardiso", "Skyline", "Frontal", "Dense"
```

In the GUI, click **▶ Run** to solve the active load case, or **▶▶ All** to batch-solve all cases.
In Python: `model.solve()` (all cases) or `model.solve(load_case=1)` (single case).

---

## Linear Backends

### 1. Intel Pardiso (Primary Sparse Solver)
The primary high-performance solver for large-scale parallel sparse systems. Integrated via FFI from Intel oneAPI MKL [5, 28].

- **Unsymmetric (Robust)**: Default mode (mtype 11). Suitable for any well-posed problem.
- **Symmetric Optimization (Fast)**: Enabled via `symmetric_optimization = true`. Uses mtype 2 (SPD), resulting in ~10x speedup for symmetric systems like grids and trusses.
- **Requirement**: Intel MKL libraries must be available.

### 2. Skyline Solver (Efficiency Baseline)
A highly optimized profile-storage solver implemented in pure Rust.

- **Advantage**: Zero external dependencies and extremely low overhead for small to medium problems (~5000 DOFs).
- **Complexity**: $O(N \cdot B^2)$ where $B$ is the bandwidth.
- **Usage**: Recommended for 2D models and smaller 3D structures.

### 3. Frontal Solver (Memory Optimized)
A solver designed for massive models by processing the global matrix in "fronts" to minimize active memory footprint [27, 30].

- **Status**: Currently implemented as a research structure. Slower than Skyline for small models due to active memory management.

### 4. Dense Solver (Legacy/Small Scale)
Standard LU decomposition on a full square matrix.

- **Advantage**: Extremely robust for debugging small elements.
- **Disadvantage**: $O(N^3)$ complexity makes it unusable for models larger than ~1000 DOFs.

### 5. SuiteSparse
Integration with UMFPACK via `russell_sparse`.

- **Status**: Feature-gated (`suitesparse`). Primarily targeted at MacOS environments where MKL is unavailable.

## Configuration (TOML)

Solvers are configured in the `[config]` section of your project file:

```toml
[config]
solver = "Pardiso"              # Linear backend: "Pardiso", "Skyline", "Frontal", "Dense"
symmetric_optimization = true   # Massive speedup for Pardiso if system is SPD
```

## Solver Selection Guide

| Model Size | Recommended Backend | Notes |
| :--- | :--- | :--- |
| < 100 DOFs | Dense or Skyline | Simple, no setup |
| 100 – 5,000 DOFs | Skyline | Pure Rust, fast for banded systems |
| 5,000+ DOFs | Pardiso | Parallel sparse, best for large models |
| MacOS (no MKL) | SuiteSparse | Feature-gated alternative |
