# Element Reference

FEAP supports a variety of structural elements compatible with the FENDA input format.

## Overview

Elements are defined in the `[[elements]]` array. Each element must reference a `material` and optionally a `section`.

| Type | Nodes | DOFs/Node | Description |
| :--- | :--- | :--- | :--- |
| `Truss` | 2 | 3 (Transl.) | Pin-jointed uniaxial rod (Axial only) |
| `Beam` | 2 | 6 (Full) | Timoshenko Beam (Axial, Shear, Bending, Torsion) |

---

## Truss Element (`Truss`)

The `Truss` element is a 2-node uniaxial element that transmits only axial forces. It assumes pin-jointed connections.

### Schema
```toml
[[elements]]
type = "Truss"
nodes = [1, 2]
material = 1
area = 0.005  # Optional heritage override
```

### Supported Loads
- **Dead Load**: Self-weight based on material density `rho`.
- **Temperature**: `delta_t` causes thermal expansion/contraction.
- **Nodal Loads**: Forces applied directly to the connected nodes.

---

## Beam Element (`Beam`)

The `Beam` element is a 2-node element based on Timoshenko beam theory, which includes shear deformation. It is the primary element for frame analysis.

### Schema
```toml
[[elements]]
type = "Beam"
nodes = [3, 4]
material = 1
section = 2     # Reference to [sections]
up = [0, 1, 0]  # Local y-axis orientation
```

### Property Calculations
If a `section` is provided, FEAP automatically calculates:
- **Area (A)**: Cross-sectional area.
- **Inertia (Iy, Iz)**: Moments of inertia about local axes.
- **Torsion (J)**: St. Venant torsion constant.
- **Shear (ky, kz)**: Timoshenko shear correction factors.

### Supported Loads
- **Distributed Loads**: Force per unit length (`q`).
- **Thermal Gradient**: `delta_t` (axial) and `gradient = [Ty, Tz]` (bending).
- **Dead Load**: Gravity loads on element mass.

---

## Cross-Sections

Sections are defined in the `[sections]` block and shared across elements.

### Rectangle
```toml
[sections]
1 = { type = "Rectangle", width = 0.2, height = 0.4 }
```

### Circle / Pipe
```toml
2 = { type = "Circle", radius = 0.1 }
3 = { type = "Pipe", radius = 0.1, thickness = 0.01 }
```

### Generic (Manual Properties)
```toml
4 = { type = "Generic", area = 0.01, i_y = 1e-4, i_z = 2e-4, j = 1.5e-4 }
```
