# Mathematical Basis

The mathematical and mechanical foundations of the FEAP engine are based on established continuum mechanics theories and variational principles, mirroring those applied in the historical FENDA system [1]. 

This chapter provides an overview of the essential equations underlying global matrix assembly, element formulation, and non-linear solution algorithms.

## 1. Principle of Virtual Displacements (Variational Principle)

The basis for deriving the finite element equations in FEAP is the principle of virtual displacements, also known as the **Rayleigh-Ritz** variational principle [2, 3]. The equilibrium of a volume element requires that the sum of internal virtual strain energy and the external virtual work done by body and surface forces equals zero [3, 4].

Expressed in terms of virtual strain \\( \delta\epsilon \\), stress tensor \\( \sigma \\), body force vector \\( b \\), and surface traction vector \\( t \\), this yields:
\\[ \int_V \delta\epsilon^T \sigma \, dV - \int_V \delta u^T b \, dV - \int_\Gamma \delta u^T t \, d\Gamma = 0 \\] [3, 4]

Using the approximation for nodal displacements \\( u \approx N \cdot a \\) (where \\( N \\) represents the shape functions and \\( a \\) the nodal degrees of freedom), this integral equation transforms into the classical linear system of equations:
\\[ K \cdot a = f \\] [4, 5]
Here, \\( K \\) is the global stiffness matrix and \\( f \\) is the global load vector.

## 2. Isoparametric Element Formulation (2D Continua)

With the introduction of surface elements (`Quad4`, `Quad8`), FEAP utilizes an isoparametric element formulation. This means that the exact same shape functions \\( N_i \\) are used to map both the element geometry and the displacement field:
\\[ x = \sum N_i \cdot x_i, \quad y = \sum N_i \cdot y_i, \quad u = \sum N_i \cdot a_i \\] [6, 7]

### The Jacobian Matrix and Numerical Integration
To calculate the strains (derivatives of displacements with respect to the global coordinates \\( x, y, z \\)), the integration domain is transformed from the global system to a local, natural coordinate system \\( (r, s, t) \\) using the Jacobian matrix \\( J \\) [6]:
\\[ 
J = \begin{bmatrix}
\frac{\partial x}{\partial r} & \frac{\partial y}{\partial r} & \frac{\partial z}{\partial r} \\\
\frac{\partial x}{\partial s} & \frac{\partial y}{\partial s} & \frac{\partial z}{\partial s} \\\
\frac{\partial x}{\partial t} & \frac{\partial y}{\partial t} & \frac{\partial z}{\partial t}
\end{bmatrix} 
\\] [6]

The integration of the stiffness matrices is not performed analytically, but numerically via Gauss quadrature, where the function is evaluated at discrete integration points and multiplied by weight factors \\( H \\) [7]:
\\[ \int_{-1}^1 \int_{-1}^1 f(r,s) \, \text{det}J \, dr \, ds \approx \sum_{i=1}^n \sum_{j=1}^m H_i H_j f(r_i, s_j) \\] [7]

## 3. Non-Linear Solution Methods (Newton-Raphson)

In cases of material non-linearity (elasto-plasticity) or large deformations, the stiffness becomes a function of the strains: \\( D = D(\epsilon) \\) [8, 9]. The simple linear system loses its validity, and the load must be applied iteratively.

FEAP uses the **Newton-Raphson method** for this purpose. In each load step (increment), equilibrium must be established between the external nodal loads \\( f_{ext} \\) and the internal nodal forces \\( f_{int} \\) resulting from element stresses. The residual \\( R \\) must converge to zero:
\\[ R = f_{ext} - f_{int} = 0 \\]

If the residual is non-zero, the solver calculates the corrective deformations \\( \Delta u \\) using the tangential stiffness matrix (Jacobian matrix) \\( K_T \\):
\\[ K_T \cdot \Delta u = R \\]

**Total Load and State Inheritance:**
Instead of calculating pure load increments as in classical approaches, FEAP utilizes a *Total Load Approach*. If a load case inherits a converged state (stresses, displacements, plastic strains), the solver calculates the difference to the new absolute target load and smoothly interpolates this external load vector \\( f_{ext} \\) over the specified load steps.

## 4. Stress States and Yield Conditions

To describe and limit material strength, three-dimensional stress states are often described not by individual components, but by the invariants of the stress tensor and the stress deviator tensor [10, 11]. 

The most important yield and failure criteria in FEAP (such as the BiLinearSteel model or the upcoming geotechnical models) define their yield surface \\( f(\sigma) = 0 \\) in principal stress space (e.g., according to von Mises or Mohr-Coulomb) [12, 13]:
*   **Elastic domain:** \\( f(\sigma) < 0 \\) [14]
*   **Plastic flow:** \\( f(\sigma) = 0 \\) [14]

A stress state outside the yield surface (\\( f(\sigma) > 0 \\)) is physically impossible and is projected back onto the yield surface as part of the non-linear constitutive law (**Return-Mapping**) [15].
