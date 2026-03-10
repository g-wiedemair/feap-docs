# Potential Fields (Heat Transfer & Seepage)

FEAP is not strictly limited to structural deformation analysis. Following the theoretical foundation of the historical FENDA system, the engine's architecture generalizes perfectly to solve scalar potential field problems. 

The mathematical formulation for these problems is identical, whether calculating **steady-state heat conduction** (Fourier's Law) or **groundwater seepage** in geotechnical engineering (Darcy's Law).

## 1. Governing Differential Equations

Let \\( \Phi \\) represent the scalar potential field (e.g., temperature \\( T \\) or hydraulic head \\( h \\)). The transfer rate (flux) \\( q \\) of the physical quantity is proportional to the potential gradient:
\\[ q = -k \nabla \Phi \\]
where \\( k \\) is the matrix of conductivity (thermal conductivity or soil permeability) [4].

From the principle of conservation of energy or mass, the internal equilibrium requires:
\\[ \nabla^T q - Q = 0 \quad \Rightarrow \quad \nabla^T k \nabla \Phi + Q = 0 \\]
where \\( Q \\) represents an internal source or sink term (e.g., heat generation or water pumping) [4].

## 2. Boundary Conditions

To solve the differential equation, two types of boundary conditions can be applied [4]:
1.  **Dirichlet Boundary (Essential):** The potential itself is prescribed on the boundary \\( \Gamma_\Phi \\):
    \\[ \Phi = \bar{\Phi} \\]
2.  **Neumann / Robin Boundary (Natural):** The normal component of the transfer rate is defined on the boundary \\( \Gamma_q \\):
    \\[ q_n = \bar{q} + \alpha \Phi \\]
    where \\( \bar{q} \\) is a constant specified flux and \\( \alpha \\) is a convection or radiation coefficient.

## 3. Finite Element Formulation

Using the Rayleigh-Ritz variational principle, the problem is cast into minimizing the functional \\( \Pi \\) over the domain \\( \Omega \\):
\\[ \Pi = \frac{1}{2} \int_\Omega (\nabla \Phi)^T k (\nabla \Phi) \, d\Omega - \int_\Omega Q \Phi \, d\Omega + \frac{1}{2} \int_{\Gamma_q} \alpha \Phi^2 \, d\Gamma + \int_{\Gamma_q} \bar{q} \Phi \, d\Gamma \\]

By introducing the finite element discretization \\( \Phi \approx \sum N_i a_i \\) (where \\( N_i \\) are the shape functions and \\( a_i \\) are the nodal potential degrees of freedom), the minimization condition \\( \partial \Pi / \partial a = 0 \\) yields the linear system of equations [2]:
\\[ H \cdot a + f = 0 \\]

Here, the conductivity matrix \\( H \\) and the generalized load vector \\( f \\) are assembled numerically using Gauss quadrature:
\\[ H_{ij} = \int_\Omega (\nabla N_i)^T k (\nabla N_j) \, d\Omega + \int_{\Gamma_q} \alpha N_i N_j \, d\Gamma \\]
\\[ f_i = -\int_\Omega N_i Q \, d\Omega + \int_{\Gamma_q} N_i \bar{q} \, d\Gamma \\]

## 4. Implementation in FEAP

Because potential field analysis requires only one degree of freedom (DOF) per node (the scalar value \\( \Phi \\)), it is computationally much cheaper than structural analysis. 
In FEAP, the 2D isoparametric continuum elements (`Quad4`, `Quad8`) introduced for plane stress/strain can be seamlessly utilized as 2D potential elements (analogous to the historic `FI2` element). The engine assigns a single active DOF, integrates the \\( H \\) matrix using the exact same Jacobian \\( J \\) transformations, and solves the resulting system using the standard sparse linear solvers (e.g., Intel Pardiso).
