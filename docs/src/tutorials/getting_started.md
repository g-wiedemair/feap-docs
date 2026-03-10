# Tutorial 1: Getting Started with FEAP

This tutorial guides you through creating and analyzing your first finite element model using FEAP. We will build the **Linear Static Truss** example (`01_truss.toml`), which matches the reference output from the legacy FENDA software.

## 1. Project Structure

FEAP uses TOML files to define models. A typical model file contains:
- `[config]`: System type and solver settings.
- `[nodes]`: Geometry definitions.
- `[materials]`: Material properties.
- `[sections]`: Cross-section definitions.
- `[elements]`: Connectivity.
- `[[load_cases]]`: Load definitions.
- `[[combinations]]`: Load combinations.

## 2. Defining the Model (`01_truss.toml`)

Create a new file named `01_truss.toml` and follow along.

### Configuration
First, we define the system type. For a 2D truss, we use `Truss2D`. This optimizes the solver by ignoring rotational degrees of freedom.

```toml
[config]
system_type = "Truss2D"
solver = "Pardiso" # Uses the high-performance Intel Pardiso solver
description = "Reference Truss 9 Nodes / 15 Elements"
```

### Nodes
Define the geometry. The format is `ID = [X, Y, Z]`.
Our truss has a span of 8.0m and height of 1.5m.

```toml
[nodes]
# Bottom Chord (y=0)
1 = [0.0, 0.0, 0.0]
2 = [2.0, 0.0, 0.0]
3 = [4.0, 0.0, 0.0] # Middle Node
# ... (add nodes 4, 5)

# Top Chord (y=1.5)
6 = [1.0, 1.5, 0.0]
# ... (add nodes 7, 8, 9)
```

### Boundary Conditions
Fix the supports.
- `1 = [true, true]` fixes Ux and Uy (Pin).
- `5 = [false, true]` fixes Uy only (Roller).

```toml
[boundary_conditions]
1 = [true, true]  # Pin Support
5 = [false, true] # Roller Support
```

### Materials & Sections
Define Steel S355 and a Circular section.

```toml
[materials]
1 = { type = "LinearElastic", name = "Steel S355", E = 2.1e11, nu = 0.3, rho = 7850.0 }

[sections]
1 = { type = "Circle", radius = 0.05 } # r=50mm
```

### Elements
Connect the nodes using `Truss` elements. You can assign parts for grouping.

```toml
[elements]
# Bottom Chord (Part 1)
1 = { type = "Truss", nodes = [1, 2], material = 1, section = 1, part = 1 }
# ...
```

## 3. Defining Loads

FEAP supports multiple load cases and combinations.

### LC1: Dead Load (Self-Weight)
Automatic gravity load based on material density `rho`.

```toml
[[load_cases]]
id = 1
name = "Dead Load"
dead_load = [0.0, -9.81, 0.0]
```

### LC2: Nodal Loads
Apply point forces. Here we apply -100kN downward at node 4.

```toml
[[load_cases]]
id = 2
name = "Nodal Loads"
[load_cases.nodal_forces]
4 = [0.0, -100000.0, 0.0] # 100 kN Down at Node 4
```

### LC3: Temperature
Apply thermal load to the bottom chord.

```toml
[[load_cases]]
id = 3
name = "Temperature"
[load_cases.temperature_loads]
1 = { delta_t = 20.0 }
```

### Combinations
Create a Design Combination (ULS).
Equation: `1.35 * LC2 + 1.0 * LC3`

```toml
[[combinations]]
id = 4
name = "ULS Combination"
factors = [ [2, 1.35], [3, 1.0] ]
```

## 4. Running the Analysis

Run FEAP from the terminal or using the GUI.

```bash
cargo run --release -- examples/01_truss.toml
```

## 5. Visualizing Results

Open the GUI. You can:
- Select **Load Case 4** (ULS Combination) from the header dropdown.
- View **Displacements** (scaled arrows).
- Check **Reactions** at supports.
- Inspect **Axial Forces** (N) in the elements.

In this example, the mid-span displacement at Node 3 for the combination case should be approximately **1.2 mm**.
