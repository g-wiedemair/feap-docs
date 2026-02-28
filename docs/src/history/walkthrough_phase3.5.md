# System Update: Performance, Tests, Docs & Camera

This update addressed four key areas of the FEAP project (Phase 3.5).

## 1. Performance Upgrade (Collections)
- **Change**: Replaced `std::collections::HashMap` with `hashbrown::HashMap` across `feap_core`.
- **Hasher**: Configured `ahash` (AES-NI accelerated) for O(1) lookups.
- **Impact**: Faster assembly and solver performance, especially for large models.
- **Files**: `feap_core/src/collections.rs`, `feap_core/Cargo.toml`.

## 2. Test Suite Restructuring
- **Change**: Moved unit tests from `src/tests.rs` to dedicated integration tests in `tests/`.
- **New Structure**:
  - `feap_core/tests/assembly_tests.rs`: Sparse matrix assembly validation.
  - `feap_core/tests/linear_solver_tests.rs`: Linear and Non-linear solver tests (including snap-through).
- **Benchmarks**: Added `feap_core/benches/solver_bench.rs` using `criterion` to measure assembly and solve times.

## 3. Living Documentation
- **Change**: initialized `docs/` folder with `mdBook` structure.
- **Content**:
  - `introduction.md`: Project overview.
  - `architecture.md`: High-level design.
  - `performance.md`: Optimizations and benchmarks.
  - `user_guide.md`: Usage and input format.

## 4. Camera Controller (GUI)
- **Change**: Implemented robust Arcball/Orbit camera controls.
- **Controls**:
  - **Middle Mouse (MMB)**: Orbit (rotate around target).
  - **Shift + MMB**: Pan (move view plane).
  - **Scroll**: Zoom (dolly in/out).
- **Implementation**: Uses spherical coordinates for stability (no gimbal lock or drift).

## Verification
- **Build**: `cargo build --workspace` succeeds.
- **Tests**: `cargo test -p feap_core` passes (5 tests).
- **Docs**: `docs/` structure is ready (requires `mdbook` to serve).
