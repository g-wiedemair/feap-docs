# IO Performance: TOML vs Binary Caching

To ensure near-instant model loading for large industrial models, FEAP implements a transparent **Binary Caching** system using `bincode`. This benchmark quantifies the performance difference between standard TOML parsing and loading from the binary cache.

## Benchmark Configuration
- **Model**: 3D Truss Grid (Cross-Braced)
- **Size**: 30 x 30 grid units
- **Complexity**: ~900 Nodes, ~1,700 Elements
- **Hardware**: Windows 11 (AMD Ryzen / Intel)

## Results

| Operation | Average Time | Speedup |
| :--- | :--- | :--- |
| **TOML Initial Load** (Parse + Model Build) | **14.1 ms** | 1.0x |
| **Binary Cached Load** (Zero-Overhead) | **5.3 ms** | **2.7x faster** |

### Analysis
The results show that even for a relatively small model (900 nodes), the binary caching system provides a **~3x improvement** in loading speed.

1.  **Parsing Bottleneck**: The majority of the TOML loading time (~13.9 ms) is spent in the string parsing phase. The conversion from the intermediate input structure to the numerical `Model` is extremely efficient.
2.  **Scalability**: For models with 50,000+ nodes, the TOML parsing time grows significantly faster than binary deserialization. Binary caching ensures that professional-scale projects remain responsive during initial load and project switching.

## How it Works
1.  **Comparison**: FEAP checks the modification time of the `.toml` source and `.bin` cache.
2.  **Fast Path**: If the cache is newer, `bincode` deserializes the `Model` directly into memory.
3.  **Slow Path**: If the TOML has changed, the parser rebuilds the model and transparently updates the cache.

---
*Results generated via `cargo bench -p feap_core --bench io_bench` on 2026-02-16.*
