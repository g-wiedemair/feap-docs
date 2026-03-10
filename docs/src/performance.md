# Solver Performance Comparison

This report compares the performance of different linear system solvers in FEAP across various scale and element types, demonstrating the efficiency of our Rust-based assembly and the Intel Pardiso FFI integration.

## 1. Structural Grid Benchmark (30x30 Truss, ~1,800 DOFs)

A classic 2D framework test to evaluate the baseline overhead of the solvers.

| Solver Backend | Matrix Type | Time (ms) | Speedup (vs Skyline) |
| ------ | ------ | ------ | ------ |
| **Skyline** | Profile | 4.8 | 1.0x (Baseline) |
| **Pardiso Fast** | Symmetric (mtype 2) | 10.1 | 0.5x |
| **Pardiso Robust** | Unsymmetric (mtype 11) | 111.2 | 0.04x |
| **Frontal** | Frontal | 320.9 | 0.015x |
| **Dense** | Full Square | 1368.0 | 0.003x |

*Analysis:* For small, regular 2D grids with extremely low bandwidth, the specialized pure-Rust **Skyline** solver remains the fastest due to zero external setup overhead. The $O(N^3)$ complexity of the **Dense** solver makes it nearly 300x slower, proving it is strictly for debugging small models. 

## 2. Large Scale 2D Continuum (Plate with Hole, ~24,500 DOFs)

This benchmark evaluates a linear elastic 2D continuum utilizing quadratic `Quad8` elements. The system yields 24,480 active equations after boundary conditions are applied. The test measures a full Newton-Raphson load step including global matrix assembly.

| Solver Backend | Matrix Type | Time (s) | Speedup (vs Skyline) |
| ------ | ------ | ------ | ------ |
| **Pardiso Fast** | Symmetric (mtype 2) | **0.754** | **1.7x** |
| **Pardiso Robust** | Unsymmetric (mtype 11) | 1.001 | 1.3x |
| **Skyline** | Profile | 1.301 | 1.0x (Baseline) |

*Analysis:* 
1. **The Sparse Advantage:** As the model scales into the tens of thousands of DOFs, the parallel **Intel Pardiso** solver takes the clear lead. Solving a highly complex 24.5k DOF linear system in just 0.75 seconds enables near real-time feedback during engineering design.
2. **Symmetric Optimization:** The difference between Pardiso Unsymmetric (1.00s) and Pardiso Symmetric (0.75s) highlights the efficiency of extracting and factoring only the upper-triangular matrix (mtype 2) for standard structural problems.
3. **Skyline Scalability:** With a mean semi-bandwidth of only $B = 227$, the Skyline solver still performs remarkably well (1.3s) due to its $O(N \cdot B^2)$ scaling, remaining a highly capable fallback for users without Intel MKL installed.

## Recommendations

1. Use **Skyline** for small to medium-sized 2D models (up to ~5,000 DOFs) due to zero setup overhead.
2. Enable **`symmetric_optimization = true`** and use **Pardiso** for large 3D models or 2D continua (> 5,000 DOFs) to fully utilize multi-core sparse matrix factorization.
3. Use **Pardiso Robust** (default) only if the system is known to be unsymmetric or non-positive-definite.
4. Avoid **Frontal** and **Dense** for any production-scale models.