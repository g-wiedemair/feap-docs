# Load Definition Reference

FEAP provides a flexible loading system to apply forces, moments, and thermal gradients to your models. Loads are always grouped inside **Load Cases**, which manage the non-linear application of these forces during the Newton-Raphson iteration.

## Nodal Loads

Point forces and point moments applied directly to specific nodes in the global \\( X, Y, Z \\) directions.
*   **Forces (fx, fy, fz):** Applicable to all element types.
*   **Moments (mx, my, mz):** Only applicable if the node possesses rotational degrees of freedom (e.g., nodes connected to an `Beam`).

### TOML Example
```toml
[[load_cases]]
id = 1
name = "Dead Load"

[[load_cases.nodal_loads]]
node = 15
fy = -150.0
mz = 25.0
Python Example
lc1 = model.add_load_case(id=1, name="Dead Load")
lc1.add_nodal_load(node_id=15, fy=-150.0, mz=25.0)

1D Element Loads (Beams & Trusses)
Loads distributed along the length of 1D structural elements.
Uniform Distributed Load
A constant force per unit length (e.g., self-weight, snow). It can be applied in the global coordinate system or the element's local system.
TOML Example
[[load_cases]]
id = 2
name = "Snow Load"

[[load_cases.uniform_loads]]
element = 10
qy = -10.0
Python Example
lc2 = model.add_load_case(id=2, name="Snow Load")
lc2.add_uniform_load(element_id=10, qy=-10.0)


2D Continuum Loads (Edge Loads)
For 2D elements (Quad4, Quad8), external pressures like earth pressure, wind, or surface surcharges must be applied to the boundaries of the mesh. FEAP handles this via Edge Loads (force per unit length).
The engine internally integrates this continuous line load using the element's exact isoparametric shape functions N 
i
​
  and converts it into mathematically consistent nodal force vectors. This is particularly crucial for quadratic Quad8 elements, where mid-side nodes receive a different proportion of the total force than corner nodes.
Application: Instead of manually selecting specific element edges, you define a path of nodes outlining the boundary. The engine automatically detects the corresponding element faces.
TOML Example
[[load_cases]]
id = 3
name = "Earth Pressure"

[[load_cases.edge_loads]]
name = "Right Wall"
path = [1-4]
qx = -50.0
parts = [5]
Python Example
lc3 = model.add_load_case(id=3, name="Earth Pressure")
lc3.add_edge_load_by_path(
    name="Right Wall",
    path_nodes=[1-4], 
    qx=-50.0, 
    qy=0.0, 
    parts=[5]
)
