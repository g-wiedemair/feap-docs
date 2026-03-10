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
- `system_type`: String ID (e.g., "Frame2D", "Truss3D", "PlaneStress", "General3D").
- `description`: A brief project description.
- `solver`: Backend choice ("Dense", "Skyline", "Frontal", "Pardiso", "SuiteSparse").
- `symmetric_optimization`: Boolean. Enable for Pardiso if system is symmetric positive definite.

### `set_length_unit(unit: str)`
Sets the length unit for the model (e.g. "m", "mm").

### `set_output(path=None, echo_input=None, echo_geometry=None, displacements=None, forces=None, reactions=None, precision=None)`
Configures automatically generated output files (.out).
- `displacements`/`forces`/`reactions`: Can be "ALL", "NONE", or a list of specific IDs.

### `save(path: str)` / `to_toml()`
Saves the current model to a TOML file or returns the TOML string.

### `load(path: str)` -> `Model`
Static method to load a model from a TOML file.

### `write_results(path: str)`
Writes a formatted text report of the current results.

### Model Metrics (Utility)
- `node_count()`: Number of nodes.
- `element_count()`: Number of elements.
- `total_volume()`: Total volume of all elements.
- `total_mass()`: Total mass (Volume * Rho).
- `get_max_plastic_strain()`: Utility for monitoring plastic failure.

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

### `add_circular_section(id: int, radius: float)`
Adds a solid circular cross-section.

### `add_pipe_section(id: int, radius: float, thickness: float)`
Adds a pipe (hollow circular) cross-section.

### `add_generic_section(id: int, area: float, iy: float, iz: float, j: float, shear_y=None, shear_z=None)`
Adds a generic cross-section with manually specified properties.

---

## Geometry

### `add_node(id: int, x: float, y: float, z: float=0.0)`
Adds a node. Z defaults to 0.0 for 2D systems.

### `fix_node(id: int)`
Fully fixes a node (all 6 DOFs).

### `set_bc(id: int, ux: bool, uy: bool, uz: bool=None, rx: bool=None, ry: bool=None, rz: bool=None)`
Sets specific boundary conditions. True = fixed, False = free.

### `add_beam(id: int, n1: int, n2: int, mat_id: int, ...)`
Adds a 3D Beam element. 
- **Tapering**: Supports tapering (linear property variation) if lists of size 2 are provided for `area`, `iy`, `iz`, `j`, or `section`.
- `section`: Optional ID(s) of a pre-defined cross-section.

### `add_truss(id: int, n1: int, n2: int, mat_id: int, ...)`
Adds a 3D Truss element. Supports tapering.

### `add_quad4(id: int, nodes: list[int], material_id: int, thickness: float=1.0)`
Adds a 4-node quadrilateral continuum element.

### `add_quad8(id: int, nodes: list[int], material_id: int, thickness: float=1.0)`
Adds an 8-node quadratic quadrilateral continuum element.

### `apply_node_imperfection(node_id: int, dx: float, dy: float, dz: float)`
Applies a direct geometric imperfection (coordinate shift) to a node.

### `apply_imperfection(mode_shape: dict, max_amplitude: float)`
Scales a buckling mode shape and applies it as a global geometric imperfection.

---

## Loads & Load cases

### `Model.add_load_case(id: int, name: str)` -> `LoadCase`
Defines a new load case and returns a `LoadCase` object for modification.

### `Model.add_combined_load_case(id: int, name: str, factors: list[tuple])`
Defines a combination (superposition) of existing results.
`factors`: List of `(case_id, factor)` tuples.

### `LoadCase` Class Methods
The following methods are called on the object returned by `add_load_case`.

- `add_nodal_force(node_id, fx=0.0, fy=0.0, fz=0.0)`
- `add_dead_load(gx, gy, gz)`: Set gravity/accelerations.
- `add_uniform_load(element_id, intensity, direction, projected=False, system="Global")`
- `add_trapezoidal_load(element_id, q1, q2, direction, projected=False)`
- `add_distributed_load(element_id, locations, values, direction, projected=False)`
- `add_temperature_load(elements, delta_t, gradient_y=0.0, gradient_z=0.0)`: `elements` can be int or list.
- `add_edge_load_by_path(path_nodes, qx, qy, parts=None, elements=None)`: Distributes traction along element boundaries.
- `add_edge_load(element_id, edge_index, qx, qy)`: Direct application to an element face (0: bottom, 1: right, 2: top, 3: left).
- `continue_from(other_load_case)`: Inherits the converged physical state for construction stage sequencing.
- `clear_loads()`: Removes all definitions from this case.

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
Returns the 6-DOF displacement vector `[ux, uy, uz, rx, ry, rz]`.

### `get_reaction(node_id: int, load_case=1)` -> `list[float]`
Returns the reaction forces `[Fx, Fy, Fz, Mx, My, Mz]` at a support.

### `get_element_forces(element_id: int, rel_pos=1.0, load_case=None)` -> `list[float]`
Returns internal forces `[N, Vy, Vz, Mt, My, Mz]` at relative position `rel_pos` (from 0.0 to 1.0).
Returns only `[N]` for Truss elements.

### `get_buckling_mode(mode_index: int)` -> `dict[int, list[float]]`
Returns a dictionary mapping node IDs to their 6-DOF mode shape vectors.
`mode_index` starts at 1.

