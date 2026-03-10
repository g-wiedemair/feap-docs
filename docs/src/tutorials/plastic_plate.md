# Non-Linear Material Modeling (Plastic Plate with a Hole)

In this tutorial, we will utilize the non-linear features of FEAP. We will procedurally generate a mesh for a plate with a central hole using quadratic 8-node elements (`Quad8`). We will load it beyond its yield strength to induce plasticity, and subsequently define a second load case to unload it, allowing us to investigate permanent (residual) deformations. 

This example demonstrates two core concepts of the FEAP engine:
1. **Full State Inheritance:** Sequential load cases can inherit the exact converged physical state (displacements, total strains, stresses, and plastic history) of a previous step.
2. **Total Load Approach:** You always define the *absolute target load* for a given load case. If a load case inherits from a previous state, the solver automatically and smoothly interpolates the external forces from the inherited starting state to your new target state over the specified `load_steps`.

## 1. The Python Script

Create a file named `plate_hole_plasticity.py` and paste the following code:

```python
import math
import feap

model = feap.Model()

# Set config
model.set_config(system_type="PlaneStress", description="Plate with hole", solver="Pardiso", symmetric_optimization=True)
model.set_solver_nonlinear(max_iterations=50, load_steps=10)

# 1. Define material (Structural Steel)
model.add_elastoplastic_material(
    id=1, e=210000.0, nu=0.3, rho=7.85e-9, fy=235.0, et=2100.0, name="Steel S235"
)

# 2. Geometry parameters
r_hole = 10.0       
width = 50.0        
n_r = 90            
n_t = 60            
thickness = 5.0     
load = 100.0 * 5    

# 3. Generate nodes
def get_node_id(i, j, n_t_val):
    return i * (2 * n_t_val + 1) + j + 1

edge_nodes = []
for i in range(2 * n_r + 1):
    xi = (i / (2 * n_r))  
    for j in range(2 * n_t + 1):
        eta = j / (2 * n_t) 
        theta = eta * (math.pi / 2.0)

        x_in = r_hole * math.cos(theta)
        y_in = r_hole * math.sin(theta)
        
        if theta < math.pi / 4.0:
            x_out = width
            y_out = width * math.tan(theta)
        else:
            y_out = width
            x_out = width / math.tan(theta) if theta != (math.pi / 2.0) else 0.0

        x = (1.0 - xi) * x_in + xi * x_out
        y = (1.0 - xi) * y_in + xi * y_out
        node_id = get_node_id(i, j, n_t)

        if i % 2 == 1 and j % 2 == 1:
            continue
        model.add_node(node_id, x, y)

        # 4. Boundary conditions
        if j == 0:
            model.set_bc(node_id, ux=False, uy=True)
        elif j == 2 * n_t:
            model.set_bc(node_id, ux=True, uy=False)

        if i == 2 * n_r and abs(x_out - width) < 1e-5:
            edge_nodes.append(node_id)

# 6. Generate elements
elem_id = 1
for i in range(n_r):
    for j in range(n_t):
        n1 = get_node_id(2*i, 2*j, n_t)
        n2 = get_node_id(2*i+2, 2*j, n_t)
        n3 = get_node_id(2*i+2, 2*j+2, n_t)
        n4 = get_node_id(2*i, 2*j+2, n_t)
        n5 = get_node_id(2*i+1, 2*j, n_t)
        n6 = get_node_id(2*i+2, 2*j+1, n_t)
        n7 = get_node_id(2*i+1, 2*j+2, n_t)
        n8 = get_node_id(2*i, 2*j+1, n_t)
        model.add_quad8(elem_id, [n1, n2, n3, n4, n5, n6, n7, n8], material_id=1, thickness=thickness, part=1)
        elem_id += 1

# 7. Define Sequential Load Cases
# Load Case 1: Pulling the plate to the target load
lc1 = model.add_load_case(id=1, name="Step 1: Loading")
lc1.add_edge_load_by_path(path_nodes=edge_nodes, qx=load, qy=0.0, generate_path=False, parts=[1], name="Right Edge Pull")

# Load Case 2: Unloading the plate
lc2 = model.add_load_case(id=2, name="Step 2: Unloading")
# Inherit the fully converged and plastified physical equilibrium from LC1
lc2.continue_from(lc1)
# Define the new absolute target load. Setting qx=0.0 tells the solver to smoothly 
# ramp down the external forces from 'load' to 0.0 over the specified load_steps.
lc2.add_edge_load_by_path(path_nodes=edge_nodes, qx=0.0, qy=0.0, generate_path=False, parts=[1], name="Right Edge Unload")

# 8. Save and Solve
model.save("06_plate_quad8.toml")
model.solve()
```

## 2. Evaluation in the GUI

### Step 1 (Loading)
Switch to the first load case. Select the "Equivalent Plastic Strain" view. You will clearly see that the elements at the top and bottom of the hole have yielded due to stress concentration.

### Step 2 (Unloading)
Switch to the second load case. Select the "Displacement Magnitude" view and activate "Show Deformed". Because the solver started from the inherited equilibrium of Step 1 and smoothly reduced the external load to zero, the plate exhibits permanent residual deformation and self-equilibrating stresses trapped around the hole.
