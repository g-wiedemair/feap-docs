# Phase 9: Python Integration & Scripting

## Hot-Reloading ✅
- [x] **File Monitoring**: Integrated `notify` crate for filesystem event tracking.
- [x] **Background Watcher**: Implementation of an asynchronous subscription channel in `iced` to monitor the active `.toml` file.
- [x] **Live Sync**: Automatic model reload and viewport update on external file save.
- [x] **QoL**: Robust handling of non-atomic saves and momentary file locks.

## Config & Output Exposure ✅
- [x] **Global Settings**: Python API for `SystemType` and `SolverType` configuration.
- [x] **Output Control**: Selection of nodes/elements for results export in scripting.
- [x] **Serialization**: Fixed TOML serialization for configuration and output settings.

## Convenience & Smart Defaults ✅
- [x] **2D API**: Simplified coordinates (optional z) and nodal forces.
- [x] **Smart BCs**: System-aware boundary condition defaults (e.g., auto-fixing out-of-plane DOFs).
- [x] **Default Propagation**: Automatic application of smart defaults during node creation in Python.

## GUI Project Workflow ✅
- [x] **Load Script Menu**: Added "Load Script" to the File menu with `Ctrl+L` shortcut.
- [x] **Feedback**: Integrated log counters and badges for script execution monitoring.

## Materials & Sections ✅
- [x] **Descriptions**: Support for material names and IDs in scripting.
- [x] **Section Bridge**: Implementation of `add_rectangular_section`, `add_circular_section`, etc.
- [x] **Property Precedence**: Elements automatically derive properties from sections if provided.

## Serialization & Robustness ✅
- [x] **System-Aware Export**: TOML serialization optimized for 2D/3D systems (omitting redundant data).
- [x] **Flexible Parsing**: Robust handling of padded vectors and coordinates in the input layer.

## Results & UX Sync ✅
- [x] **Dynamic Validation**: GUI automatically detects if script results are valid for visualization.
- [x] **Scaling Logic**: Synchronized deformation scales and automatic tab switching after solve.

## Loads & Physics Bridge ✅
- [x] **Dead Load**: Python API for gravity modeling.
- [x] **Distributed Loads**: Uniform, Trapezoidal, and Generalized line loads.
- [x] **Temperature Loads**: Support for multi-element $\Delta T$ and thermal gradients.

## Multi-Loadcase Results & Reactions ✅
- [x] **ResultSet Infrastructure**: Core support for independent results per load case.
- [x] **Batch Solver**: Automatic evaluation of all primary load cases and combinations.
- [x] **Reaction Engine**: Robust R = Ku - F calculation for support nodes.
- [x] **Python API**: Enhanced getters with load case support (`get_reactions`, `get_displacement`).
- [x] **GUI Sync**: Automated result validation and scale synchronization after scripting runs.
