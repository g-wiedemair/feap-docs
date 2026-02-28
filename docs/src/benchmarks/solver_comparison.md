# Solver Performance Comparison

This report compares the performance of different linear system solvers in FEAP for a 30x30 2D Truss grid (~1800 Degrees of Freedom).

## Latest Results (2D Truss Grid - 30x30)

| Solver Backend | Matrix Type | Iterations | Time (ms) | Speedup (vs Skyline) |
|:--- |:--- |:--- |:--- |:--- |
| **Skyline** | Profile | 1 | 4.8 | 1.0x (Baseline) |
| **Pardiso Fast** | Symmetric (mtype 2) | 1 | 10.1 | 0.5x |
| **Pardiso Robust** | Unsymmetric (mtype 11) | 1 | 111.2 | 0.04x |
| **Dense** | Full Square | 1 | 1368.0 | 0.003x |
| **Frontal** | Frontal | 1 | 320.9 | 0.015x |

### Analysis

1.  **Skyline Efficiency**: For regular 2D grids with low bandwidth, the specialized Skyline solver remains the fastest due to minimal overhead and $O(N)$ efficiency within the profile.
2.  **Symmetric Optimization**: `Pardiso Fast` is **~11x faster** than `Pardiso Robust`. This confirms that using `mtype 2` and only assembling the upper triangular part of the matrix significantly reduces the computational burden for symmetric systems.
3.  **Frontal Solver Optimization**: The Frontal solver was refactored to use symbolic analysis and pre-allocated buffers. This eliminated the previous `resize()` bottleneck, improving performance from >10s to **~320ms**. While still slower than Skyline for this specific bandwidth, it is now competitive for large-scale analysis.
4.  **Pardiso Overhead**: For small problems (~1800 DOFs), Pardiso's setup overhead (symbolic factorization, multi-threading initialization) makes it slightly slower than a tuned Skyline solver, though it will likely dominate as problem size increases.
5.  **SparseAssembler Impact**: The transition to `SparseAssembler` and `CsMat` has brought `Pardiso Robust` from minutes down to ~111ms, a massive overall performance gain.
6.  **Dense Limitations**: The $O(N^3)$ complexity of the Dense solver makes it unsuitable for grids larger than 20x20.
 At 1800 DOFs, it is nearly **300x faster** than the Dense solver.

### Dense Solver
The Dense solver is suitable only for small models. Its performance degrades quadratically/cubically as the model size increases.

### Frontal Solver
The Frontal solver uses a wave-front method to eliminate degrees of freedom as soon as they are fully summed. The optimized implementation uses symbolic analysis to pre-allocate memory once, making it competitive for large models (320ms for 30x30 truss).

### Pardiso & SuiteSparse
- **Pardiso**: Fully integrated via Intel oneAPI MKL. 
    - **Robust Mode**: Uses `mtype 11` (Unsymmetric) and full CSR matrix. Reliable for any system but carries overhead for full matrix assembly (~111ms).
    - **Fast Mode**: Uses `mtype 2` (SPD) and extracts only the upper triangular part. Achieves **~10x speedup** over Robust mode, becoming competitive with Skyline even at small scales.
- **SuiteSparse**: Currently not supported on Windows due to `suitesparse-src` build complexities. Planned for future Linux-based builds.

## Recommendations
1. Use **Skyline** for small to medium-sized 2D models (up to ~5000 DOFs) due to zero setup overhead.
2. Enable **`symmetric_optimization = true`** and use **Pardiso** for large 3D models or complex grids where the bandwidth of the Skyline solver becomes a bottleneck.
3. Use **Pardiso Robust** (default) only if the system is known to be unsymmetric or non-positive-definite.
4. Avoid **Frontal** and **Dense** for any production-scale models.
