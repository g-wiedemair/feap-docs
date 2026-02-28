# User Guide

This guide covers the basic usage of FEAP, from preparing input files to visualizing results.

## Editor IntelliSense (Auto-completion)

FEAP provides a JSON Schema for its TOML input files. This enables auto-completion, validation, and tooltips in modern editors like VS Code.

To enable it in **VS Code**:
1. Install the **Even Better TOML** extension.
2. Add this line to the top of your `.toml` input file:

```toml
#:schema https://raw.githubusercontent.com/g-wiedemair/feap/main/docs/feap-schema.json
```

*(Note: While developing locally, you can also point to the local file path: `#:schema ./docs/feap-schema.json`)*

## Running the Application

### GUI Mode (Interactive 3D Editor)
To launch the full graphical interface for editing and visualization:
```bash
cargo run --release -p feap_gui
```
You can also open a specific file directly:
```bash
cargo run --release -p feap_gui -- examples/01_truss.toml
```

### Console Mode (Batch Processing)
For headless analysis (e.g., on a server or for scripts):
```bash
cargo run --release -p feap_console -- input.toml
```

### Hot-Reloading (Real-Time Sync)
FEAP features built-in **Hot-Reloading** for project files. If you open your project's `.toml` file in an external editor (like VS Code), FEAP will automatically detect whenever you save the file.

The application instantly reloads the model data and updates the 3D viewport without requiring you to click any buttons or restart the session. This provides a seamless "Live Coding" experience for iterating on structural geometry, materials, and loads.

## GUI Manual

### Navigation
- **Orbit**: Hold `Middle Mouse Button` (MMB) and drag.
- **Pan**: Hold `Shift + MMB` and drag.
- **Zoom**: Scroll `Mouse Wheel`.
- **Auto-Fit**: Camera automatically centers on the model on load.

### Header Controls
The top bar provides essential controls:

- **Load Case Selector**: Use the `‹` and `›` buttons or the dropdown menu to switch between Load Cases (e.g., "Dead Load") and Combinations (e.g., "ULS").
    - The **active load case** determines which forces are drawn and which results are visualized.
    - When switching, the deformation scale automatically adjusts to the new case's displacement magnitude.
- **▶ Run** (blue/green): Solves only the **currently active** load case using the Newton-Raphson solver. Turns green once results are available.
- **▶▶ All** (gray): Solves **all** primary load cases and combinations in batch using Newton-Raphson. Each case is solved independently.
- **Window Controls**: Minimize, Maximize, and Close buttons (integrated for borderless look).

### Menu Bar
The secondary menu provides quick access to common actions:

| Action | Description |
| :--- | :--- |
| **New / Open / Close** | Project management |
| **Save / Save As** | Save the current TOML project |
| **Solve Current** | Same as ▶ Run — solves active load case |
| **Solve All** | Same as ▶▶ All — batch solve all cases |
| **Reset Camera** | Re-centers the viewport on the model |
| **Show Results / Show Model** | Toggle between model and results view |
| **Scale** | Set the deformation scale factor |

### Inspector (Right Sidebar)
The right sidebar controls view settings and displays object data.

#### Visualization
- **Color By**: Switch the coloring mode of the mesh.
    - **Part**: Colors elements based on their `part` ID (useful for grouping).
    - **Material**: Colors elements by Material ID.
    - **Section**: Colors elements by Cross-Section ID.
- **Deformation**:
    - **Show Deformed**: Toggles the display of the deflected shape.
    - **Scale**: Sets the multiplier for displacement visualization. Auto-calculated on solve to be ~10% of model extent.

#### Results View
When results are available, the inspector shows:
- **Displacements** (Ux, Uy, Uz, Rx, Ry, Rz) for selected nodes.
- **Reaction Forces** at constrained nodes.
- **Internal Forces** (N, Vy, Vz, Mt, My, Mz) for selected elements.
- **Stresses**: Selectable stress components for 3D beam extrusions (available in the "Result Field" dropdown):
    - **Normal Stress**: Combined axial and bending stress.
    - **Shear Stress**: Combined shear and torsion.
    - **Von Mises Stress**: Equivalent stress calculated as $\sigma_V = \sqrt{\sigma_{xx}^2 + 3\tau^2}$.
- **Plastic Strain**: For fiber-based beam elements, visualizes the magnitude of permanent deformation. High plastic strain values (red) indicate the formation of plastic hinges.

#### Selection
Click on any node or element in the viewport to inspect its properties (ID, Coordinates, Connectivity, Results) in the **Selection** card.

### Left Sidebar
The left sidebar shows an overview of the model:
- **Materials** list with assigned colors.
- **Cross-Sections** list.
- **Parts** list with visibility toggles.
- **Reference** section with copyable TOML snippets for common patterns.

### Console
The bottom panel shows application logs and provides scripting capabilities.
- **Level Filter**: Toggle `ERROR`, `WARN`, `INFO`, `DEBUG`, `TRACE` to control verbosity.
- **Deps**: Toggle `Deps` to show/hide internal engine logs (wgpu, iced).
- **Load Script**: Click to load and execute a Python `.py` script. See [Python Tutorial](tutorials/python_tutorial.md).

### Python Scripting
FEAP includes an embedded Python engine for parametric modeling and automation. Scripts can:
- Generate geometry programmatically (nodes, elements, materials).
- Define loads, boundary conditions, and load cases.
- Trigger the solver and extract results.
- **Elasto-Plasticity**: Define non-linear materials and safety factors.
    ```python
    model.add_elastoplastic_material(
        id=1, E=210e6, nu=0.3, rho=7.85, 
        yield_strength=235e3, tangent_modulus=2100.0, 
        gamma_m=1.1, name="Steel S235"
    )
    ```

For details, see the **[Python Tutorial](tutorials/python_tutorial.md)** and the **[Elasto-Plastic Buckling Tutorial](tutorials/01_elastoplastic_buckling.md)**.

## Input Format
FEAP uses a structured TOML format incompatible with legacy FENDA decks but conceptually similar.

For a step-by-step guide on creating your first model, see the **[Getting Started Tutorial](tutorials/getting_started.md)**.

For detailed element and load definitions, refer to the **[Element Reference](reference/elements.md)** and **[Loads Reference](reference/loads.md)**.
