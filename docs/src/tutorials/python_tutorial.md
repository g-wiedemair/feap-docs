# Tutorial 2: Introduction to Python Scripting

FEAP provides a robust, embedded Python API via `PyO3`. This allows you to bypass the graphical interface and TOML files entirely, enabling parametric modeling, automated optimization loops, and batch processing directly from your Python environment.

In this introductory tutorial, we will build a very basic linear-elastic truss structure, define a load case using the object-oriented API, and solve it.

## 1. Setting up the Model

First, import the `feap` module and initialize an empty `Model`. You can also set basic configuration parameters like the system type and a description.

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
FEAP uses an object-oriented approach to manage load cases. You instantiate a load case via the model and then apply loads directly to this specific case. This makes managing multiple load scenarios extremely clean.

```python
# Create a primary load case
lc1 = model.add_load_case(1, "Dead Load")
# Apply a load to this specific load case
# Enable global gravity (Y-direction)
lc1.add_dead_load(0, -9.806, 0)
```

FEAP allows you to easily combine mechanical and thermal loads.

```python
lc2 = model.add_load_case(2, "Joint load")
# Apply nodal force at midspan (fz defaults to 0.0)
mid_node = (num_bays // 2) + 1
lc2.add_nodal_force(mid_node, 0.0, -500.0)

lc3 = model.add_load_case(3, "Temperature Load")
lc3.add_temperature_load([1, 5, 9, 13, 17, 21], 20.0)
```

You can also combine existing load cases.

```python
model.add_combined_load_case(4, "ULS", [(1, 1.35), (2, 1.5), (3, 1.3)])
```

## 5. Solving, Extracting Results and Saving
Finally, you can trigger the internal Newton-Raphson solver and save the generated model to a .toml file.

### Initialize Logging
```python
# Initialize logging to see the solver output in the console
feap.init_logging()
```

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
    disp = model.get_displacement(node_id=3, load_case=4)
    print(f"Node 3 - Ux: {disp[0]:.6f}, Uy: {disp[1]:.6f}")
    
    # 2. Extract Reaction Forces at Node 1 → returns [Fx, Fy, Fz, Mx, My, Mz]
    react = model.get_reactions(node_id=1, load_case=4)
    print(f"Support Node 1 - Fx: {react[0]:.2f}, Fy: {react[1]:.2f}")
    
    # 3. Extract Internal Forces for Element 2 at midspan
    #    Returns [N, Vy, Vz, Mt, My, Mz] (depends on element type)
    forces = model.get_element_forces(elem_id=2, rel_pos=0.5, load_case=4)
    print(f"Normal Force in Element 2: {forces[0]:.2f}")
```

### Saving the Model

```python
# Save the model to a .toml file
model.save("my_model.toml")
```


## 6. Running the Script

### From the console
```bash
python script_name.py
```

### From the GUI
1. Open FEAP and load any project (or start empty).
2. Click **Load Script** in the console area.
3. Select your `.py` file.
4. The 3D viewport instantly updates with your geometry.
5. Click **▶ Run** or **▶▶ All** to solve (or call `model.solve()` in the script).

### Hot-Reloading
If you edit your `.py` script in an external editor and save, FEAP automatically detects the change and re-executes the script — providing a live-coding workflow.

