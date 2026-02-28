# Performance

FEAP is optimized from the ground up for large-scale finite element simulations [Performance chapter].

## Key Optimizations
- **Hashbrown & ahash**: Internal hash tables utilize `ahash` (AES-NI accelerated) for O(1) lookups during critical **DOF mapping** during assembly [Performance chapter].
- **Sparse Matrix Infrastructure**: The global stiffness matrix is stored efficiently in the **Compressed Sparse Row (CSR)** format via the `sprs` crate [Performance chapter].
- **Parallel Assembly**: Computation of element stiffness matrices is parallelized across all available CPU cores using **Rayon** [4, 5].
- **High-Performance Solvers**: Integration with **Intel Pardiso** enables the parallel solution of sparse linear systems on multi-core hardware [5].
- **Binary Caching**: Transparently caches processed models as `.bin` files via `bincode`, resulting in **~3x faster loading** for large projects [IO Performance (TOML vs Binary)](benchmarks/io_bench.md).

## Benchmarking
We utilize `criterion` to verify performance and detect regressions.

*   **Solver Comparison**: Comprehensive analysis of Dense, Skyline, and Pardiso backends [Solver Performance (30x30 Grid)](benchmarks/solver_comparison.md).

```bash
cargo bench -p feap_core
```
Current benchmarks focus on the assembly speed of truss chains and the solution efficiency of linear systems [Performance chapter].
