# Loads Reference

FEAP supports multiple load types organized into **Load Cases** and **Combinations**. Each load case defines a set of forces, and combinations allow you to superimpose them with safety factors.

## Load Cases

Load cases are defined as `[[load_cases]]` arrays in the TOML input file. Each case gets a unique `id` and optional `name`.

```toml
[[load_cases]]
id = 1
name = "Dead Load"
```

### Nodal Forces
Point forces and moments applied directly to nodes. Format: `node_id = [Fx, Fy, Fz, Mx, My, Mz]`.

For 2D systems, you can use the shortened form `[Fx, Fy]`.

```toml
[[load_cases]]
id = 2
name = "Point Loads"
[load_cases.nodal_forces]
3 = [0.0, -100000.0, 0.0]     # 100 kN downward at Node 3
7 = [50000.0, 0.0, 0.0]       # 50 kN horizontal at Node 7
```

### Distributed Loads
Force per unit length applied to beam elements. Supported parameters vary by load direction:

```toml
[[load_cases]]
id = 3
name = "Traffic"
[load_cases.distributed_loads]
# Element ID → list of load objects
1 = [{ qy = -5000.0 }]                     # Uniform load in local y
2 = [{ qy = -3000.0 }, { qx = 1000.0 }]    # Multiple loads on same element
```

| Parameter | Description |
| :--- | :--- |
| `qx` | Axial distributed load (N/m) |
| `qy` | Transverse load in local y (N/m) |
| `qz` | Transverse load in local z (N/m) |

### Dead Load (Self-Weight)
Gravity load computed automatically from material density `rho` and the cross-section area. Specify the gravity acceleration vector:

```toml
[[load_cases]]
id = 1
name = "Self-Weight"
dead_load = [0.0, -9.81, 0.0]   # g in [m/s²], applied in global Y
```

For 2D: `dead_load = [0.0, -9.81]`

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

| Parameter | Description |
| :--- | :--- |
| `delta_t` | Uniform temperature change (°C/K) |
| `gradient` | `[Ty, Tz]` — Temperature gradient causing bending |

The thermal strain is: `ε_th = α · ΔT`, where `α` is the thermal expansion coefficient defined in the material.

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

> **Note**: When using the Newton-Raphson solver with **▶▶ All**, each combination is solved independently as a full nonlinear analysis (not by superposition). This correctly handles geometric nonlinearities.

---

## Python API

Load cases can also be defined programmatically:

```python
# Create load case
model.add_load_case(id=1, name="Dead Load")
model.set_dead_load(0.0, -9.81)

# Add forces in the active load case
model.add_nodal_force(3, 0.0, -50000.0)
model.add_distributed_load(1, 0.0, -5000.0)
model.add_temperature_load(2, 30.0)

# Create combination
model.add_combination(id=10, name="ULS", factors=[(1, 1.35), (2, 1.50)])
```

See the [Python Tutorial](../tutorials/python_tutorial.md) for complete examples.
