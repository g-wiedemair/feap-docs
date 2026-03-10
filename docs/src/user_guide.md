# User Guide: Interactive 3D Editor

FEAP provides a modern, GPU-accelerated graphical user interface (GUI) built with `wgpu` and `iced`. It is designed for both interactive model inspection and professional results evaluation.

## Input Format & Editor IntelliSense (Auto-completion)

FEAP uses a structured TOML format for its input files (`.toml`). While it is conceptually similar to legacy FENDA decks, it provides a much cleaner, modern syntax [5].

FEAP provides a JSON Schema for these TOML input files. This enables auto-completion, validation, and tooltips in modern editors like VS Code [1].
To enable it in **VS Code**:
1. Install the **Even Better TOML** extension.
2. Add this line to the very top of your `.toml` input file:
```toml
#:schema https://github.com/g-wiedemair/feap-docs/blob/main/docs/feap-schema.json
```
(Note: While developing locally, you can also point to the absolute local file path).


## GUI Manual

### Navigation

*   **Orbit**: Hold `Middle Mouse Button` (MMB) and drag.
*   **Pan**: Hold `Shift + MMB` and drag.
*   **Zoom**: Scroll `Mouse Wheel`.
*   **Auto-Fit**: Double click middle mouse button. The camera automatically centers on the model.

### Header Controls & Heads-Up Display (HUD)

The top bar provides essential controls for your analysis session:

*   **Load Case Selector**: Use the `‹` and `›` buttons or the dropdown menu to switch between Load Cases. The active load case determines which forces are drawn and which results are visualized.
*   **▶ Run (blue/green)**: Solves only the currently active load case using the Newton-Raphson solver. Turns green once results are valid.
*   **▶▶ All (gray)**: Solves all primary load cases and combinations in batch.

The HUD overlays critical analysis information directly on the viewport, preventing the need to dig through text logs:

*   Total reaction forces `[Fx, Fy, Fz]` at the supports.
*   Buckling Mode factors $\lambda$ (if an eigenvalue analysis was performed).

### Menu Bar & Workspaces

The secondary menu provides quick access to common actions:

*   **File Menu**: Seamlessly manage projects via `New`, `Open`, `Open Recent`, `Save`, and `Close`.
*   **Load Script**: Load and execute a Python script directly into the session.
*   **Standard Views**: Quickly align the camera to `Top`, `Bottom`, `Front`, `Back`, or `Isometric` perspectives.

### Left Sidebar & Console

*   **Left Sidebar**: Shows an overview of the model including `Materials`, `Cross-Sections`, and a `Parts` list with visibility checkboxes to quickly show/hide elements.
*   **Console**: Located at the bottom, showing application logs. Use the `Level Filter` to toggle `ERROR`, `WARN`, `INFO`, `DEBUG`, and `TRACE` logs to control verbosity.

### Right Sidebar (Inspector) & Rendering Modes

Click on any node or element in the viewport to inspect its properties (ID, Coordinates, Connectivity, Results) in the Inspector. The Inspector also allows you to toggle different visualization modes:

*   **Solid / Wireframe Mode**: Switch between shaded elements and wireframe representation.
*   **Color Modes**:
    *   **Part**: Colors elements based on their assigned structural part.
    *   **Material**: Colors elements based on their material definition.
    *   **Section**: Colors elements based on their geometric cross-section.
*   **Results (Heatmaps)**: Visualize continuous heatmaps for Displacements, Strains, and Stresses. For 2D continua (`Quad4`/`Quad8`), stresses are accurately smoothed across element boundaries using advanced **Nodal Recovery**.
*   **Ghost Mode**: Displays a transparent, undeformed silhouette of the structure alongside the deformed shape for easy visual comparison.

### Python Scripting & "Dry-Run" Mode

FEAP includes an embedded Python engine for parametric modeling, non-linear elasto-plasticity setups, and automation. You can load a `.py` script via the `File -> Load Script` menu or the button in the Console.

**Dry-Run (Import-Only) Mode:**
When you load a script that contains the `model.solve()` command, the GUI intelligently sets an internal environment variable (`FEAP_GUI_IMPORT_MODE = 1`). This tells the Python engine to **bypass the solver execution** during the import phase.

*   **Benefit**: Large, complex models are loaded and rendered instantly for visual inspection of the mesh and boundary conditions without freezing the UI.
*   **Execution**: Once you have verified the geometry visually, simply click the green **"Run Solver"** button in the header to perform the actual batch calculation.

### High-Performance Binary Caching (`.bin`)

To handle industrial-scale models smoothly, the engine automatically creates a binary cache file (`.bin`) using `bincode` serialization upon the first successful load of a TOML or Python file. Subsequent openings of the same model bypass the text/script parsing entirely, resulting in **~3x faster loading times** and a drastically reduced memory footprint during viewport interactions.
