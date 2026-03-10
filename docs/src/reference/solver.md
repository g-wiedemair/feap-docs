# Solvers & Numerical Methods

FEAP employs a stratified solver architecture. At the top level, a robust **Newton-Raphson** algorithm handles non-linear load stepping and state inheritance. For the resulting linearized tangential systems \\( K_T \cdot \Delta u = R \\), the engine delegates the matrix factorization to highly optimized, pluggable linear solver backends.

## Non-Linear Solver (Newton-Raphson)

The primary solver for static and construction stage analyses is the Newton-Raphson solver. It iteratively reduces the residual unbalance forces between external applied loads and internal element stresses.

### Convergence Criteria
The iteration process successfully terminates when the following criteria are met:
*   **Force tolerance**: The norm of the residual vector \\( ||R|| \\) falls below a defined threshold (default: `1e-8`).
*   **Displacement tolerance**: The norm of the corrective displacement increment \\( ||\Delta u|| \\) falls below a defined threshold (default: `1e-10`).
*   **Maximum Iterations**: If convergence is not reached within the specified limit (default: `50`), the solver aborts the current step to prevent infinite loops and logs a non-convergence error.

## Linear Solver Backends

The core engine supports multiple linear solver backends. The choice of backend heavily influences memory consumption and calculation speed, depending on the scale and topology of the model.

### 1. Intel Pardiso (High-Performance Sparse)
Fully integrated via the Intel oneAPI Math Kernel Library (MKL) through a fast Foreign Function Interface (FFI) bridge. This is the absolute workhorse for industrial-scale 3D models and large 2D continua.
*   **Fast Mode (`symmetric_optimization = true`)**: Extracts and factors only the upper-triangular part of the stiffness matrix (mtype 2, Symmetric Positive Definite). Achieves massive speedups (up to 10x) and is the recommended default for standard structural mechanics.
*   **Robust Mode (`symmetric_optimization = false`)**: Uses a full asymmetric CSR (Compressed Sparse Row) matrix layout (mtype 11). This is strictly required for complex material laws with non-associated flow rules that yield unsymmetric tangential stiffness matrices.

### 2. Skyline Solver (Profile Solver)
A highly optimized profile-storage solver implemented entirely in pure, safe Rust.
*   **Advantage**: Zero external C/C++ dependencies. It compiles and works out of the box on any operating system.
*   **Complexity**: Scales with \\( O(N \cdot B^2) \\), where \\( B \\) is the mean semi-bandwidth. 
*   **Usage**: Highly recommended as the default for 1D/2D structural frameworks and smaller continua up to ~5,000 DOFs, as it carries zero external setup overhead.

### 3. SuiteSparse (UMFPACK)
Integration with the popular UMFPACK solver via the `russell_sparse` crate. 
*   **Usage**: Feature-gated (`suitesparse`). Primarily targeted as a high-performance sparse alternative for macOS/Linux environments where Intel MKL is either unavailable or difficult to link natively.

### 4. Frontal Solver (Memory Optimized)
Based on the wave-front method (historically the default in the original FENDA system), eliminating degrees of freedom as soon as they are fully assembled to minimize the active RAM footprint.
*   **Status**: Currently implemented as a research structure for processing massive out-of-core systems. For in-core calculations on modern hardware, Pardiso is strictly superior.

### 5. Dense Solver
Standard LU decomposition on a full \\( N \times N \\) square matrix.
*   **Usage**: Strictly for debugging fundamental element formulations (e.g., single element tests). The \\( O(N^3) \\) complexity makes it completely unusable for systems larger than a few hundred DOFs.

---

## Configuration & Usage Examples

You can configure the active linear solver backend and the non-linear Newton-Raphson parameters either directly through the Python API or via the structured TOML input file.

### Via Python API
```python
import feap

model = feap.Model()

# 1. Global System & Linear Solver Configuration
model.set_config(
    system_type="PlaneStress", 
    solver="Pardiso",               # Options: "Pardiso", "Skyline", "Frontal", "Dense"
    symmetric_optimization=True     # Extracts only upper-triangular matrix for massive speedups (only Pardiso)
)

# 2. Non-Linear Newton-Raphson Settings
model.set_solver_nonlinear(
    max_iterations=50,              # Abort condition to prevent infinite loops
    load_steps=10                   # Number of increments to reach the total target load
)
```

### Via TOML Input File
```toml
[config]
system_type = "PlaneStress"
solver = "Pardiso"
symmetric_optimization = true

[solver_nonlinear]
max_iterations = 50
load_steps = 10
```
