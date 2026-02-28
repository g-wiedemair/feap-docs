# Python Scripting in FEAP

FEAP features a powerful, embedded Python engine. You can write Python scripts to parametrically generate complex structures, apply loads, and extract results automatically.

## 1. Setting up your IDE (VS Code)

While you can type commands directly into the FEAP GUI console, writing scripts in an editor like VS Code is much more comfortable. To get **Autocompletion (IntelliSense)** and tooltips for the `feap` module, simply place the `feap.py` stub file (provided with your FEAP installation) into your working directory. 

Write your scripts normally. FEAP will execute the code directly using its high-speed Rust core.

```python
import feap

# Initialize the model
model = feap.Model()
```

## 2. Basic Configuration
Start by defining the system type and output configuration. FEAP supports smart defaults. If you choose `Truss2D` or `Frame2D`, the Z-axis and out-of-plane rotations are locked automatically!

```python
model.set_config(system_type="Truss2D", solver="Skyline", description="My First Script")
model.set_output(precision=4, echo_input=False)
```

**Available system types**: `General`, `Truss2D`, `Truss3D`, `Frame2D`, `Frame3D`
**Available solvers**: `Pardiso`, `Skyline`, `Frontal`, `Dense`

## 3. Materials and Geometry
You can create nodes and elements using loops. Notice how we don't need to specify the Z-coordinate for 2D systems:

```python
# 1. Define Material (ID, E-Modulus, Poisson, Density, Alpha)
model.add_linear_material(1, 210000.0, 0.3, 7.85e-9, 1.2e-5)

# 2. Add Nodes: add_node(id, x, y)
model.add_node(1, 0.0, 0.0)
model.add_node(2, 5.0, 0.0)
model.add_node(3, 2.5, 2.5)

# 3. Add Boundary Conditions (id, Ux, Uy)
model.set_bc(1, True, True)    # Pinned
model.set_bc(2, False, True)   # Roller

# 4. Add Truss Elements (id, node_a, node_b, mat_id, area)
model.add_truss(1, 1, 2, 1, 15.0)
model.add_truss(2, 2, 3, 1, 10.0)
model.add_truss(3, 1, 3, 1, 10.0)
```

### Parametric Generation
The real power of Python scripting is in loops and parametric models:

```python
# Generate a 10-bay truss automatically
n_bays = 10
bay_length = 3.0

for i in range(n_bays + 1):
    model.add_node(i + 1, i * bay_length, 0.0)        # Bottom chord
    model.add_node(100 + i, i * bay_length, 2.0)       # Top chord
```

## 4. Applying Loads
FEAP allows you to easily combine mechanical and thermal loads.

```python
# Enable global gravity (Y-direction)
model.set_dead_load(0.0, -9.81)

# Add a nodal point load at Node 3 (Fx, Fy)
model.add_nodal_force(3, 10.0, -50.0)

# Add a distributed load (traffic) to Element 1
model.add_distributed_load(1, 0.0, -5.0)

# Add a temperature increase of 30 degrees to Element 2
model.add_temperature_load(2, 30.0)
```

## 5. Solving and Extracting Results
You can trigger the Newton-Raphson solver directly from your script.

### Batch Mode (solve all cases)
When called without arguments, `solve()` runs all defined load cases and combinations:

```python
if model.solve():
    print("All load cases solved!")
```

### Single Load Case Mode
To solve only a specific load case or combination by its ID:

```python
model.solve(load_case=1)   # Solve only LC1
model.solve(load_case=4)   # Solve only Combination 4
```

### Extracting Results

```python
if model.solve():
    # 1. Extract Displacement at Node 3 → returns [Ux, Uy, Uz, Rx, Ry, Rz]
    disp = model.get_displacement(3)
    print(f"Node 3 - Ux: {disp[0]:.6f}, Uy: {disp[1]:.6f}")
    
    # 2. Extract Reaction Forces at Node 1 → returns [Fx, Fy, Fz, Mx, My, Mz]
    react = model.get_reactions(node_id=1)
    print(f"Support Node 1 - Fx: {react[0]:.2f}, Fy: {react[1]:.2f}")
    
    # 3. Extract Internal Forces for Element 2 at midspan
    #    Returns [N, Vy, Vz, Mt, My, Mz] (depends on element type)
    forces = model.get_element_forces(2, rel_pos=0.5)
    print(f"Normal Force in Element 2: {forces[0]:.2f}")
    
    # 4. Extract results for a specific load case
    disp_lc2 = model.get_displacement(3, load_case=2)
    print(f"LC2 Node 3 - Uy: {disp_lc2[1]:.6f}")
```

## 6. Running the Script

### From the GUI
1. Open FEAP and load any project (or start empty).
2. Click **Load Script** in the console area.
3. Select your `.py` file.
4. The 3D viewport instantly updates with your geometry.
5. Click **▶ Run** or **▶▶ All** to solve (or call `model.solve()` in the script).

### Hot-Reloading
If you edit your `.py` script in an external editor and save, FEAP automatically detects the change and re-executes the script — providing a live-coding workflow.

## Quick Reference

| Method | Description |
| :--- | :--- |
| `model.set_config(...)` | System type, solver, description |
| `model.add_node(id, x, y, z=0)` | Create a node |
| `model.set_bc(id, ux, uy, ...)` | Set boundary conditions |
| `model.add_linear_material(...)` | Add linear elastic material |
| `model.add_truss(id, n1, n2, mat, area)` | Add truss element |
| `model.add_beam(id, n1, n2, mat, sec)` | Add beam element |
| `model.add_nodal_force(node, fx, fy, ...)` | Apply nodal force |
| `model.add_distributed_load(elem, qx, qy)` | Apply distributed load |
| `model.add_temperature_load(elem, dt)` | Apply temperature change |
| `model.set_dead_load(gx, gy, gz=0)` | Set gravity vector |
| `model.solve(load_case=None)` | Solve (all or single) |
| `model.get_displacement(node, lc=None)` | Get nodal displacements |
| `model.get_reactions(node, lc=None)` | Get support reactions |
| `model.get_element_forces(elem, rel_pos)` | Get internal forces |