File: references/bibliography.md
# Bibliography and References

This bibliography encompasses the theoretical foundations integrated into the FENDA system over 40 years, as well as citations for the modern Rust-based infrastructure.

## 1. Classical Finite Element Literature
These works form the basis for the deformation method and global stiffness matrix assembly used in FEAP.

*   **Zienkiewicz, O.C.** (1971): *The Finite Element Method in Engineering Science*. McGraw-Hill, London [1].
*   **Bathe, K.J. & Wilson, E.** (1976): *Numerical Methods in Finite Element Analysis*. Prentice-Hall, New Jersey [1].
*   **Bathe, K.J.** (1982): *Finite Element Procedures in Engineering Analysis*. Prentice-Hall, New Jersey [2].
*   **Hughes, T.J.R.** (1987): *The Finite Element Method*. Prentice-Hall, New Jersey [3].
*   **Irons, B.M.** (1972): *A Frontal Solution Program for Finite Element Analysis*. International Journal for Numerical Methods in Engineering [2].

## 2. Nonlinear Mechanics & Stability
These references support the implementation of the **Newton-Raphson** loop and the **Arc-Length method** for post-buckling analysis.

*   **Crisfield, M.A.** (1981): *A Fast Incremental/Iterative Solution Procedure that Handles ‘Snap-Through’*. Computer & Structures Vol 13 [4].
*   **Crisfield, M.A.** (1984): *Accelerating and Damping the Modified Newton-Raphson Method*. Computer & Structures Vol 18 [5].
*   **Matthies, H. & Strang, G.** (1979): *The Solution of Nonlinear Finite Element Equations*. International Journal for Numerical Methods in Engineering [6].
*   **Maier, K.**: *Vorspannung mit Spangliedern ohne Verbund im Hochbau*. Dissertation, University of Innsbruck [4].

## 3. Geotechnics & Potential Fields
References for the implementation of 3-Phase elements and seepage flow analysis.

*   **Mualem, Y.** (1976): *A New Model for Predicting the Hydraulic Conductivity of Unsaturated Porous Media*. Water Resources Research [7].
*   **Genuchten, M.T.** (1980): *A Closed-Form Equation for Predicting the Hydraulic Conductivity of Unsaturated Soils*. Soil Science Society of America Journal [7].
*   **Öttl, G.** (2003): *A Three-Phase FE-Model for Dewatering of Soils by Means of Compressed Air*. Ph.D. Thesis, University of Innsbruck [7].

## 4. Modern Rust FEA Ecosystem
Please include the following citations in any academic or commercial work based on this project's modern Rust components.

*   **Corrado, J., Harmon, J., Notaros, B., & Ilic, M.M.** (2022): *FEM_2D: A Rust Package for 2D Finite Element Method Computations with Extensive Support for hp-refinement*. TechRxiv [8].
*   **Hovey, C. B., & Buche, M. R.** (2025): *automesh: Automatic Mesh Generation in Rust*. Journal of Open Source Software [9, 10].
*   **Rust Infrastructure**: This project utilizes `sprs` for sparse matrix management, `nalgebra` for linear algebra, `Rayon` for work-stealing parallelism, and `Intel MKL/Pardiso` for parallel sparse solving [11-13].
