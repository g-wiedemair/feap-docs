# Bibliography and References

This bibliography encompasses the theoretical foundations integrated into the FENDA system over 40 years, as well as citations for the modern Rust-based infrastructure.

## 1. Historical Reference & Baseline
*   [1] **Neuner, O.** (2000): *FENDA - Finite Element Nonlinear Design Analysis, User Manual*. Ingenieurbüro Dr. Oswald Neuner, Seefeld, Austria.

## 2. Classical Finite Element Literature
These works form the basis for the deformation method and global stiffness matrix assembly used in FEAP.
*   [3] **Zienkiewicz, O.C.** (1971): *The Finite Element Method in Engineering Science*. McGraw-Hill, London.
*   [4] **Bathe, K.J. & Wilson, E.** (1976): *Numerical Methods in Finite Element Analysis*. Prentice-Hall, New Jersey.
*   [5] **Bathe, K.J.** (1982): *Finite Element Procedures in Engineering Analysis*. Prentice-Hall, New Jersey.
*   [6] **Hughes, T.J.R.** (1987): *The Finite Element Method*. Prentice-Hall, New Jersey.
*   [7] **Irons, B.M.** (1972): *A Frontal Solution Program for Finite Element Analysis*. International Journal for Numerical Methods in Engineering.

## 3. Modern Rust FEA Ecosystem
Please include the following citations in any academic or commercial work based on this project's modern Rust components.
*   [8] **Corrado, J., Harmon, J., Notaros, B., & Ilic, M.M.** (2022): *FEM_2D: A Rust Package for 2D Finite Element Method Computations with Extensive Support for hp-refinement*. TechRxiv.
*   [9] **Hovey, C. B., & Buche, M. R.** (2025): *automesh: Automatic Mesh Generation in Rust*. Journal of Open Source Software.
*   [10] **Rust Infrastructure**: This project utilizes `sprs` for sparse matrix management, `nalgebra` for linear algebra, `rayon` for work-stealing parallelism, and `Intel MKL/Pardiso` for parallel sparse solving.
