# Phase 3.5: Performance-Refinement, Test-Infrastruktur & UX

## Aufgabe 1: Collections Performance Upgrade (hashbrown + ahash)
- [x] Add `hashbrown` + `ahash` to `feap_core/Cargo.toml`
- [x] Create `feap_core::collections` module with type aliases
- [x] Replace `std::collections::HashMap` in all 8 files
- [x] Verify build compiles cleanly

## Aufgabe 2: Test-Suite Restrukturierung & Benchmarks
- [x] Split `tests.rs` → `feap_core/tests/assembly_tests.rs` + `linear_solver_tests.rs`
- [x] Remove `src/tests.rs` and `mod tests;` from `lib.rs`
- [x] Create `benches/solver_bench.rs`
- [x] Use `criterion` for benchmarking

## Aufgabe 3: Living Documentation (mdBook)
- [x] Create `docs/` mdBook structure (`book.toml`, `SUMMARY.md`)
- [x] Integrate Architecture & Performance chapters (linking to benchmarks)
- [x] Document usage instructions in `user_guide.md`

## Aufgabe 4: CameraController Finalisierung (Blender-Navigation)
- [x] Implement MMB Orbit, Shift+MMB Pan, Scroll Zoom
- [x] Use spherical coordinates (Azimuth/Polar/Radius)
- [x] Integrate with `iced` Input-Events in `app.rs`
