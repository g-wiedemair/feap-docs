# Element Library Reference

FEAP provides a highly optimized set of finite elements for both 1D/3D structural framework analysis and 2D continuum mechanics. Every element is tightly integrated with the material system and the non-linear Newton-Raphson solver.

---

## 1D Structural Elements

1D elements are the backbone of structural framework analysis. They require the assignment of both a **Material** (via `material_id`) and a **Cross-Section** (via `section_id`).

### Truss (3D Link Element)
A basic bar element that only transmits axial forces (tension and compression).
*   **Degrees of Freedom:** 3 translational DOFs per node ($ u_x, u_y, u_z $).
*   **Capabilities:** Supports large deformations (Total/Updated Lagrange) and yielding.
*   **Supported Loads:** Nodal Forces, Uniform and Trapezoidal Distributed Loads, Dead Load (Self-weight), Temperature Gradients ($ \Delta T $).

**TOML Example:**
```toml
[elements]
1 = { type = "Truss", nodes = [1, 2], material = 1, section = 1, part = 1 }
```

**Python Example**:
```python
model.add_truss(id=1, nodes=[5, 6], material_id=1, section_id=1, part=1)
```

### Beam (Spatial Beam Element)
A robust, fully 3D beam element capable of transferring bending moments, shear forces, torsion, and axial loads.
*  **Degrees of Freedom:** 6 DOFs per node (3 translations, 3 rotations).
* **Capabilities:** 
    * Can evaluate Euler-Bernoulli (thin beams) and Timoshenko (thick beams with shear deformation) theories.
    * Includes Geometric Stiffness ($ K_g $) for explicit linear buckling and second-order (P-Delta) analysis.
    * Supports Fiber Integration for accurate elasto-plastic hinge formation.
* **Supported Loads:** Nodal Forces/Moments, Uniform and Trapezoidal Distributed Loads, Dead Load, Thermal Gradients.

**TOML Example:**
```toml
[elements]
1 = { type = "Beam", nodes = [1, 2], material = 1, section = 1, part = 1 }
```

**Python Example:**
```python
model.add_sbeam(id=2, nodes=[6, 7], material_id=1, section_id=2, part=1)
```

### Cross-Sections
To use 1D elements, you must assign a cross-section. FEAP offers two approaches: Direct Input of section properties or Parametric Shapes
.

* **Direct Input (Generic Section):** For standard steel profiles (e.g., HEA, IPE) or complex arbitrary shapes, you can directly input the exact cross-sectional area ($ A $), torsional constant ($ J $), and bending moments of inertia ($ I_y $, $ I_z $).

    ```toml
    [sections]
    1 = { type = "Generic", area = 0.01, i_y = 1e-4, i_z = 2e-4, j = 1.5e-4 }
    ```

    ```python
    model.add_generic_section(id=1, area=0.00538, iy=3.69e-5, iz=1.3e-6, it=5.0e-8)
    ```

* **Parametric Shapes:** For simple geometries, you can define the dimensions and the engine automatically calculates the required area and moments of inertia.

    * Rectangular Section:
        ```toml
        [sections]
        2 = { type = "Rectangle", width = 0.2, height = 0.4 }
        ```
        ```python
        model.add_rectangular_section(id=2, width=0.2, height=0.4)
        ```

    * Circular Section:
        ```toml
        [sections]
        3 = { type = "Circle", radius = 0.15 }
        ```
        ```python
        model.add_circular_section(id=3, r=0.15)
        ```

    * Pipe (Tubular) Section:
        ```toml
        [sections]
        4 = { type = "Pipe", radius = 0.2, thickness = 0.015 }
        ```
        ```python
        model.add_pipe_section(id=4, r=0.2, t=0.015)
        ```
---

## 2D Continuum Elements (Plane Systems)
For modeling solid plates, walls, or geotechnical soil layers, FEAP provides isoparametric 2D elements. These elements are used in PlaneStress and PlaneStrain configurations and require a thickness parameter rather than a cross-section.

### Quad4 (Linear Quadrilateral)
A 4-node quadrilateral element with linear shape functions.
* **Degrees of Freedom:** 2 translational DOFs per node ($ u_x $, $ u_y $).
* **Integration:** Standard 2×2 Gauss-Legendre quadrature.
* **Capabilities:** Fast and efficient for basic plane stress/strain. Prone to shear locking in bending-dominated scenarios (requires a finer mesh).
* **Supported Loads:** Edge Loads (distributed forces on boundaries), Dead Load, Nodal Forces.

**TOML Example:**
```toml
[elements]
1 = { type = "Quad4", nodes = [1, 32, 33, 2], material = 1, thickness = 5, part = 1 }
```

**Python Example:**
```python
model.add_quad4(id=100, nodes=[8-11], material_id=2, thickness=0.25, part=2)
```

### Quad8 (Quadratic Quadrilateral)
An 8-node quadrilateral element featuring 4 corner nodes and 4 mid-side nodes with quadratic shape functions.
* **Degrees of Freedom:** 2 translational DOFs per node ($ u_x $, $ u_y $).
* **Integration:** Exact 3×3 Gauss-Legendre quadrature.
* **Capabilities:** Highly accurate. Eliminates shear locking naturally. Ideal for stress concentrations (e.g., holes) and complex elasto-plastic yielding.
* **Node Ordering:** 4 corner nodes first, followed by the 4 mid-side nodes (counter-clockwise).
* **Supported Loads:** Edge Loads (consistently integrated over quadratic edges), Dead Load, Nodal Forces.

**TOML Example:**
```toml
[elements]
1 = { type = "Quad8", nodes = [1, 243, 245, 3, 122, 244, 124, 2], material = 1, thickness = 5, part = 1 }
```

**Python Example:**
```python
model.add_quad8(id=101, nodes=[5-7, 9, 10, 12-14], material_id=2, thickness=0.25, part=2)
```
