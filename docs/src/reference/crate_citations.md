# Crate Citations and Infrastructure

FEAP leverages a modern Rust ecosystem to achieve high numerical performance and graphics reliability [15, 16].

### Numerical Core
- **nalgebra**: Used for element-level linear algebra and dense matrix operations [17].
- **sprs**: Provides the foundational infrastructure for efficient **Compressed Sparse Row (CSR)** matrix assembly [5, 17].
- **Rayon**: Enables work-stealing parallelism for concurrent assembly of the global stiffness matrix without data races [17].
- **hashbrown & ahash**: Utilized for AES-NI accelerated $O(1)$ lookups during node and degree-of-freedom mapping [17].

### Visualization and UI
- **wgpu**: The cross-platform graphics API used for the 3D viewport, targeting Vulkan, Metal, and DirectX 12 [18].
- **Iced**: A reactive UI framework inspired by Elm, providing clean state management for the editor interface [3, 19].
- **glyphon**: Used for high-quality text rendering within the GPU-accelerated viewport [20].

### Interoperability
- **PyO3**: The planned engine for Python integration, allowing the Rust core to be controlled via a scripting console [21, 22].
- **Serde**: Powers the TOML-based input parsing and bincode serialization for project files [6, 23].
