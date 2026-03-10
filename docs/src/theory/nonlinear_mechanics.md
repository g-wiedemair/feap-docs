# Non-Linear Mechanics

FEAP is designed to handle highly non-linear structural and geotechnical problems, accommodating both material non-linearities (e.g., elasto-plasticity, yielding) and geometric non-linearities (large deformations, second-order effects).

## 1. Geometric Non-Linearity (Large Deformations)

To accurately capture large deformations and large rotations, FEAP implements two primary kinematic formulations. Both methods mathematically converge to the correct solution but differ in their reference frames and numerical suitability for specific problems [1]:

*   **Total Lagrange Formulation (T.L.F.)**: The reference frame is always tied to the **original, undeformed configuration** at $t=0$. T.L.F. is the primary choice for explicit stability analysis, snap-through buckling, and the correct evaluation of the geometric stiffness matrix ($K_g$) in spatial beams and large-deformation trusses [1, 4]. Historically, it is also strictly required for complex pipe element calculations [3].
*   **Updated Lagrange Formulation (U.L.F.)**: The reference frame is continuously updated. In each load increment, the system refers to the **last converged, deformed configuration** at time $t$. U.L.F. is the recommended standard for general, step-by-step non-linear iterations. Because the geometry is incrementally updated, it provides more robust and intuitive solutions, especially when dealing with complex elasto-plastic material laws and sequential construction stages [1, 3].

## 2. Iterative Procedures (Newton-Raphson)

To solve non-linear systems, FEAP primarily employs the **Newton-Raphson** iterative method [4]. Because the global tangential stiffness matrix \\( K_T \\) depends on the current deformation and stress state, the external load must be applied incrementally.

Within each load step, equilibrium between the external applied loads \\( f_{ext} \\) and the internal element forces \\( f_{int} \\) must be achieved. The solver iterates until the residual vector \\( R \\) approaches zero:
\\[ R = f_{ext} - f_{int} = 0 \\]

In each iteration, the tangential stiffness matrix \\( K_T \\) is assembled to compute the corrective displacement increments \\( \Delta u \\):
\\[ K_T \cdot \Delta u = R \\]

## 3. State Inheritance & Construction Stages

A cornerstone of FEAP's non-linear capabilities is **Full Physical State Inheritance** (via the `continue_from` method in the Python API). 

Unlike linear analysis, where load cases can simply be superimposed, non-linear analysis is strictly path-dependent. FEAP allows subsequent load cases to inherit the exact converged equilibrium state of a previous construction stage. This mechanism seamlessly transfers the complete physical state:
*   Total strains and nodal displacements.
*   Internal element stresses.
*   Plastic history and hardening parameters at the Gauss integration points.

This is absolutely essential for modeling complex construction sequences, deep excavations, or elasto-plastic load-unload cycles.

## 4. The Total Load Approach

When inheriting a non-linear state, FEAP utilizes a **Total Load** approach. 

Instead of requiring the user to calculate and apply manual load increments (deltas) for a new stage, the user simply defines the *absolute target load* for the end of the new construction phase. The Newton-Raphson solver automatically determines the difference between the inherited external force vector and the new target load. It then smoothly interpolates these external forces over the user-specified `load_steps` to guarantee numerical stability and correct mechanical path-dependency.

## 5. Fiber Integration Model (3D Beams)

For progressive yielding in 3D spatial beams (`SBeam`), FEAP moves beyond classical analytical cross-section properties.
*   **Numerical Integration**: Instead of using global \\( EI \\) or \\( EA \\) values, the beam's internal forces are computed by integrating stresses over a discretized fiber grid at each longitudinal Gauss point.
*   **Plastic Hinges**: This numerical approach accurately captures the gradual formation of plastic hinges as individual fibers exceed the yield threshold, allowing for a highly realistic representation of ultimate load ("Traglast") behavior.
*   **Cross-Section Discretization**: Typical sections (Rectangular, Pipe, Circular) are automatically discretized into an \\( N \times M \\) grid of fibers. Each fiber acts as an independent integration point, tracking its own physical state history (active stress and plastic strain).

## 6. Advanced Stability Solvers (Arc-Length)

For problems involving severe structural instability, such as snap-through buckling or post-critical load-deflection behavior, the standard Newton-Raphson iteration may fail at the limit point (where the stiffness matrix becomes singular). For these specialized cases, the engine supports **Arc-Length Control** (e.g., after Crisfield), which constrains the load-displacement path length, allowing the solver to navigate past limit points and track unloading branches automatically [1, 4].
