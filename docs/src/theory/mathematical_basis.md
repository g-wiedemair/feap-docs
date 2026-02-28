# Mathematical Basis

The numerical foundation of FEAP is based on the deformation method and the **Rayleigh-Ritz variation principle**, established in the original FENDA specification [9, 10].

## Rayleigh-Ritz Principle
The solution to any physical-technical problem in FEAP is found by seeking a function $u$ that makes the potential $\Pi$ stationary:
$$\delta\Pi = \delta u^T A(u) d\Omega + \delta u^T B(u) d\Gamma = 0$$

## Structural Mechanics
For structural analysis, FEAP considers the symmetric Cartesian stress tensor. The sum of external and internal virtual work is defined as:
$$\int_V \delta\epsilon^T \sigma dV - \int_V \delta u^T b dV - \int_\Gamma \delta u^T t d\Gamma = 0$$

This leads to the fundamental linear system of equations used in our numerical kernel:
$$Ka + f = 0$$
where **K** is the global stiffness matrix and **f** is the load vector [14, 15].

## 6-DOF Implementation
Following the recent core upgrade, FEAP supports up to **6 degrees of freedom per node** (3 translations, 3 rotations), ensuring compatibility between truss, beam, and shell elements.
