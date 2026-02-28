# FEAP Python Scripting API

The FEAP Python API allows for headless model generation, analysis, and result extraction. The library is implemented in Rust using PyO3 and can be imported directly in Python.

## Installation & Setup

1.  **Build the extension**: Run `maturin develop --release` inside the `feap_scripting` directory.
2.  **Import**: In your Python script, use `import feap`.

---

## Model Initialization & Configuration

### `Model()`
Creates a new, empty FEAP model.
```python
model = feap.Model()
```

### `set_config(system_type=None, description=None, solver=None, symmetric_optimization=None)`
Sets global model configuration.
- `system_type`: String ID (e.g., "Frame2D", "Truss3D", "General3D").
- `description`: A brief project description.
- `solver`: Backend choice ("Dense", "Skyline", "Frontal", "Pardiso", "SuiteSparse").
- `symmetric_optimization`: Boolean. Enable for Pardiso if system is symmetric positive definite.

### `save(path: str)`
Saves the current model (including results) to a TOML file.

### `load(path: str)` -> `Model`
Static method to load a model from a TOML file.
```python
model = feap.Model.load("bridge.toml")
```

---

## Materials & Sections

### `add_linear_material(id: int, e: float, nu: float, rho: float, alpha: float, name: str = None)`
Adds a linear-elastic material.

### `add_elastoplastic_material(id: int, e: float, nu: float, rho: float, fy: float, et: float, alpha: float=0.0, gamma_m: float=None, name: str=None)`
Adds an elasto-plastic material (Fiber Model).
- `fy`: Yield strength.
- `et`: Tangent modulus.
- `gamma_m`: Optional material safety factor.

### `add_rectangular_section(id: int, width: float, height: float)`
Adds a solid rectangular cross-section.

### `add_pipe_section(id: int, radius: float, thickness: float)`
Adds a pipe (hollow circular) cross-section.

---

## Geometry

### `add_node(id: int, x: float, y: float, z: float=0.0)`
Adds a node. Z defaults to 0.0 for 2D systems.

### `fix_node(id: int)`
Fully fixes a node (all 6 DOFs).

### `set_bc(id: int, ux: bool, uy: bool, uz: bool=None, rx: bool=None, ry: bool=None, rz: bool=None)`
Sets specific boundary conditions. True = fixed, False = free.

### `add_beam(id: int, n1: int, n2: int, mat_id: int, area=None, iy=None, iz=None, j=None, up=None, part=0, section=None)`
Adds a 3D Beam element. Supports tapering if lists of size 2 are provided for properties/sections.

### `add_truss(id: int, n1: int, n2: int, mat_id: int, area=None, part=0, section=None)`
Adds a 3D Truss element.

### `apply_imperfection(mode_shape: dict, max_amplitude: float)`
Scales a buckling mode shape and applies it as a global geometric imperfection.

---

## Loads

### `add_load_case(id: int, name: str)`
Defines a new load case.

### `add_nodal_force(node_id: int, fx=0.0, fy=0.0, fz=0.0)`
Applies a point load to a node in the active load case.

---

## Analysis & Results

### `set_solver_nonlinear(max_iterations=50, displacement_tolerance=1e-6, force_tolerance=1e-6, energy_tolerance=1e-8, load_steps=1, formulation="TLF")`
Sets non-linear solver parameters.
- `max_iterations`: Max Newton-Raphson iterations.
- `displacement_tolerance`: Convergence criterion for displacement increment norm.
- `force_tolerance`: Convergence criterion for residual (force) norm.
- `load_steps`: Number of increments to apply the full load.
- `formulation`: Kinematic formulation: "TLF" (Total Lagrange) or "ULF" (Updated Lagrange).

### `solve(load_case: int = None)` -> `bool`
Runs the Newton-Raphson nonlinear solver.

### `buckling_analysis(num_modes=1, load_case=None)` -> `list[tuple[float, dict]]`
Performs an eigenvalue buckling analysis. Returns a list of `(critical_load_factor, mode_shape_dict)`.

### `get_displacement(node_id: int, load_case=1)` -> `list[float]`
Returns the 6-DOF displacement vector for a node.

### `get_element_force(element_id: int, xi=0.0, load_case=1)` -> `list[float]`
Returns internal forces `[N, Vy, Vz, Mt, My, Mz]` at local coordinate `xi` (from -1 to 1).

