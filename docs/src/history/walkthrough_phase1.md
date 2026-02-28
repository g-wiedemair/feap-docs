# Phase 1 Walkthrough: FENDA-Rust Core

## Achievements
We have successfully implemented the MVP of the `feap_core` numerics library.

### Key Components
- **Data Structures**: `Node`, `Element` (trait), `Material` (trait), `Model`.
- **Elements**: `TrussElement` (3D Truss with stiffness matrix computation).
- **Materials**: `LinearElastic` material model.
- **Solver**: `SimpleSolver` using `nalgebra`'s LU decomposition for solving $Ku=f$.
- **Input**: TOML parser (`InputModel`) to define meshes, materials, and boundary conditions.

### Verification
A validation test `test_truss_parsing_and_solve` was implemented in `feap_core/src/input.rs`.

#### Test Case: 1D Truss
- **Nodes**:
  - Node 1: (0,0,0) Fixed.
  - Node 2: (10,0,0) Load $F_x = 10$.
- **Element**: Area $A=1.0$, Material $E=1000.0$.
- **Expected Result**: $u = \frac{FL}{EA} = \frac{10 \times 10}{1000 \times 1} = 0.1$.
- **Actual Result**: `Node 2 disp: [0.1, 0.0, 0.0]`.

### Usage
To run the verification test:
```bash
cargo test -p feap_core
```
