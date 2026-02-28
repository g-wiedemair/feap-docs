# Nonlinear Mechanics

FEAP supports both material and geometric nonlinearities, utilizing strategies refined over decades of engineering practice [16].

## Geometric Nonlinearity
We implement two primary formulations for handling large deformations:
1. **Total Lagrange Formulation (T.L.F.)**: Primary choice for stability and buckling analysis [16, 17].
2. **Updated Lagrange Formulation (U.L.F.)**: Preferred for iterative non-linear steps as it provides intuitive incremental solutions [17].

## Iterative Procedures
To solve non-linear systems, FEAP employs high-level macros that control equilibrium [18, 19]:
- **Newton-Raphson (N.R.)**: A standard iterative process where the tangent stiffness matrix is updated at every iteration [5, 18].
- **Arc-Length Method (Crisfield)**: A specialized procedure designed to pass singular points in the stiffness matrix, enabling the analysis of **snap-through problems** [5, 20, 21].

## Material Nonlinearity (Elasto-Plasticity)

FEAP implements a physically non-linear material model (`MaterialType::ElastoPlastic`) to capture the ultimate capacity of structures.

- **Constitutive Law**: The model uses a bi-linear stress-strain relationship defined by the elastic modulus ($E$), the tangent modulus ($E_t$), and the yield strength ($f_y$).
- **Return Mapping**: We employ a 1D radial return-mapping algorithm. Stresses are capped at the current yield surface, and any excess strain is converted into permanent **Plastic Strain** ($\epsilon_p$).
- **Safety Factors**: The model supports the partial safety factor $\gamma_M$ (e.g., DIN EN 1993-1-1), automatically calculating the design yield strength $f_{y,d} = f_{y,k} / \gamma_M$.

## Fiber Integration Model (3D Beams)

For progressive yielding in 3D beams (`SBeamElement`), FEAP moves beyond analytical cross-section properties.

- **Numerical Integration**: Instead of using global $EI$ or $EA$ values, the beam's internal forces are computed by integrating stresses over a discretized fiber grid at each longitudinal Gauss point. 
- **Plastic Hinges**: This numerical approach accurately captures the gradual formation of plastic hinges as individual fibers exceed the yield threshold, allowing for a realistic representation of "Traglast" (ultimate load) behavior.
- **Cross-Section Discretization**: Typical sections (Rectangle, Pipe, Circle) are automatically discretized into an $N \times M$ grid of fibers, each tracking its own history (active stress and plastic strain).

## Stability Analysis
Stability can be calculated as:
- **Bifurcation Problem**: Determining the distance to the buckling load via eigenvalue analysis (Linear Buckling).
- **Non-Linear Ultimate Load**: Iteratively calculating the physical collapse load considering geometric nonlinearity (P-Delta), imperfections, and material yielding (Elasto-Plasticity).
