# Loads Reference

FEAP supports multiple load types organized into **Load Cases** and **Combinations**. Each load case defines a set of forces, and combinations allow you to superimpose them with safety factors.

## Load Cases

Load cases are defined as `[[load_cases]]` arrays in the TOML input file. Each case gets a unique `id` and optional `name`.

```toml
[[load_cases]]
id = 1
name = "Dead Load"
```
**Python Example**
```python
lc1 = model.add_load_case(id=1, name="Dead Load")
```

## Dead Load (Self-Weight)
Gravity load computed automatically from material density `rho` and the cross-section area. Specify the gravity acceleration vector:

**TOML Example**
```toml
[[load_cases]]
id = 1
name = "Self-Weight"
dead_load = [0.0, -9.81, 0.0]   # g in [m/s²], applied in global Y
```
For 2D: `dead_load = [0.0, -9.81]`

**Python Example**
```python
lc1 = model.add_load_case(id=1, name="Dead Load")
lc1.add_dead_load(0, -9.806, 0)
```

## Nodal Forces
Point forces and point moments applied directly to specific nodes in the global $ X, Y, Z $ directions.
*   **Forces (fx, fy, fz):** Applicable to all element types.
*   **Moments (mx, my, mz):** Only applicable if the node possesses rotational degrees of freedom (e.g., nodes connected to an `Beam`).

For 2D systems, you can use the shortened form `[Fx, Fy]`.

**TOML Example**
```toml
[[load_cases]]
id = 2
name = "Point Loads"
[load_cases.nodal_forces]
3 = [0.0, -100000.0, 0.0]     # 100 kN downward at Node 3
7 = [50000.0, 0.0, 0.0]       # 50 kN horizontal at Node 7
```
**Python Example**
```python
lc1 = model.add_load_case(id=2, name="Dead Load")
lc1.add_nodal_load(node_id=15, fy=-150.0, mz=25.0)
```

## 1D Element Loads (Beams & Trusses)
Loads distributed along the length of 1D structural elements.

### Uniform Distributed Load
A constant force per unit length (e.g., self-weight, snow). It can be applied in the global coordinate system or the element's local system.

*   **Coordinate System (`system`):** Choose between `"Global"` (default) or `"Local"`.
*   **Projected Loads (`projected`):** If `true`, the load intensity is applied to the horizontal projection of the element. This is essential for snow loads on inclined roofs.

**TOML Example**
```toml
[[load_cases]]
id = 3
name = "Traffic & Snow"
[load_cases.distributed_loads]
# Element 1: Uniform load in global Y (Snow on projection)
1 = [{ qy = -2.0, projected = true }]

# Element 2: Local y-axis (e.g., wind pressure)
2 = [{ qy = -5.0, system = "Local" }]
```
**Python Example**
```python
# Snow load on projection
lc2.add_uniform_load(element_id=1, intensity=-2.0, direction=[0, -1, 0], projected=True)

# Wind load in local y
lc2.add_uniform_load(element_id=2, intensity=-5.0, direction=[0, 1, 0], system="Local")
```

### Trapezoidal & General Distributions
For loads that vary along the length of an element (e.g., hydrostatic pressure), you can define custom distributions.

**TOML Example**
```toml
[[load_cases.distributed_loads]]
# Trapezoidal load from -5.0 at start to -10.0 at end
10 = [{ qy = [-5.0, -10.0] }]

# General distribution via interpolation points
11 = [{ 
    distribution = { locations = [0.0, 0.4, 1.0], values = [0.0, -10.0, -5.0] },
    qy = 1.0 
}]
```

**Python Example**
```python
# Trapezoidal: q1, q2
lc2.add_trapezoidal_load(element_id=10, q1=-5.0, q2=-10.0, direction=[0, -1, 0])

# General: locations [0..1], values
lc2.add_distributed_load(element_id=11, locations=[0.0, 0.4, 1.0], values=[0.0, -10.0, -5.0], direction=[0, -1, 0])
```

### Temperature Loads
Thermal effects on elements. Supports uniform temperature change and thermal gradients for beam elements.

```toml
[[load_cases]]
id = 4
name = "Summer Temperature"
[load_cases.temperature_loads]
# Truss: uniform expansion/contraction
1 = { delta_t = 30.0 }

# Beam: uniform + gradient (bending)
5 = { delta_t = 20.0, gradient = [10.0, 0.0] }
```
```python
lc3 = model.add_load_case(3, "Temperature Load")
lc3.add_temperature_load([1, 5, 9, 13, 17, 21], 20.0)
```

| Parameter | Description |
| :--- | :--- |
| `delta_t` | Uniform temperature change (°C/K) |
| `gradient` | `[Ty, Tz]` — Temperature gradient causing bending |

The thermal strain is: `ε_th = α · ΔT`, where `α` is the thermal expansion coefficient defined in the material.

---

## 2D Continuum Loads (Edge Loads)
For 2D elements (Quad4, Quad8), external pressures like earth pressure, wind, or surface surcharges must be applied to the boundaries of the mesh. FEAP handles this via Edge Loads (force per unit length).
The engine internally integrates this continuous line load using the element's exact isoparametric shape functions $ N_i $ and converts it into mathematically consistent nodal force vectors. This is particularly crucial for quadratic Quad8 elements, where mid-side nodes receive a different proportion of the total force than corner nodes.

* **Path-Based Application:** Define a path of nodes outlining the boundary. The engine automatically detects the corresponding element faces.
* **Direct Application:** Apply load directly to a specific element and edge index.

**Direct Edge Load (TOML)**
```toml
[[load_cases.edge_loads]]
element_id = 100
edge_index = 2   # 0: bottom, 1: right, 2: top, 3: left
qx = 0.0
qy = -10.0
```

**Direct Edge Load (Python)**
```python
# element_id, edge_index (0-3), qx, qy
lc3.add_edge_load(100, 2, 0.0, -10.0)
```

**TOML Example**
```toml
[[load_cases]]
id = 3
name = "Earth Pressure"

[[load_cases.edge_loads]]
name = "Right Wall"
path = [1-4]
qx = -50.0
parts = [5]
```

**Python Example**
```python
lc3 = model.add_load_case(id=3, name="Earth Pressure")
lc3.add_edge_load_by_path(
    name="Right Wall",
    path_nodes=[1-4], 
    qx=-50.0, 
    qy=0.0, 
    parts=[5]
)
```

---

## Load Combinations

Combinations allow you to superimpose primary load cases with factors. They are defined as `[[combinations]]` arrays.

```toml
[[combinations]]
id = 10
name = "ULS Design"
factors = [ [1, 1.35], [2, 1.50], [3, 0.90] ]
```

The `factors` list contains `[load_case_id, factor]` pairs. The combination result is:

$$S_{comb} = \sum_i \gamma_i \cdot S_i$$

### Example: Eurocode ULS
```toml
# Ultimate Limit State: 1.35·G + 1.50·Q
[[combinations]]
id = 10
name = "ULS - Eq. 6.10"
factors = [ [1, 1.35], [2, 1.50] ]

# Serviceability Limit State: 1.0·G + 1.0·Q
[[combinations]]
id = 11
name = "SLS - Characteristic"
factors = [ [1, 1.0], [2, 1.0] ]
```
```python
model.add_combined_load_case(4, "ULS", [(1, 1.35), (2, 1.5), (3, 1.3)])
```

---

## Total Load Approach & Load Inheritance
If a non-linear analysis inherits the converged physical state of a previous load case, FEAP utilizes a **Total Load** approach. This is essential for sequentially modeling construction stages, excavation, or cyclic loading.

* **TOML (`inherit_from`):** Specify the name of the parent load case.
* **Python (`continue_from`):** Pass the previous `LoadCase` object to the new case.
You do not define load increments (deltas). Instead, you define the absolute target load for the end of the new construction stage. The Newton-Raphson solver automatically computes the difference between the inherited external force vector and the new target, smoothly interpolating the load over the specified load_steps to ensure path-dependent numerical stability.