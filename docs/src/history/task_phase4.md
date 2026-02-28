# Phase 4 Task List: SBEAM, Solver-Finalisierung & Stability

## Aufgabe 1: Technischer Meilenstein SBEAM (6-DOF & Beam Element)
- [x] Upgrade `feap_core` to 6-DOF System (Node, Assembler `[isize; 6]`)
- [x] Implement `SBeamElement` (Linear Axial/Torsion, Cubic Bending)
- [x] Implement Timoshenko Factors ($\Phi$) for shear effects
- [x] Implement Geometric Stiffness Matrix ($K_g$) for Buckling
- [x] Verify Compatibility with Truss (expand to 12x12 locally)
- [x] Integration Tests: Axial, Bernoulli, Timoshenko, $K_g$ (4/4 passed)

## Aufgabe 2: Finalisierung der Solver-Suite (Skyline & Frontal)
- [x] Implement `SkylineSolver` (Profile Storage + In-memory LDLt)
- [x] Implement `FrontalSolver` structure (Disk-based Logic stub)
- [x] Implement Runtime Solver Dispatch (`SolverType` enum)

## Aufgabe 3: Log-Bereinigung & wgpu-Stability
- [x] Fix `D3D12_RESOURCE_STATE_PRESENT` / Semaphore Error (Review Surface Config)
- [x] Configure Logger to filter `wgpu`/`naga` noise (Set to WARN)
- [x] Fix `winit` log noise

## Aufgabe 4: Living Documentation (Phase 4)
- [x] Document SBEAM formulation in `docs/`
- [x] Create Theoretical Foundations chapters
- [x] Document 6-DOF input format in User Guide
- [x] Create SBeam Buckling Showcase (Benchmark)
