# Material Models Reference

FEAP provides a diverse library of constitutive material models to simulate both structural and geotechnical behavior. The material model defines the relationship between strains and stresses $ \sigma = D(\epsilon) $ and dictates how the tangential stiffness matrix $ K_T $ is updated during the non-linear Newton-Raphson iteration.

Every structural and continuum element must be assigned a valid material via its `material_id`.

## Linear Elasticity (Hooke's Law)

The standard model for linear-elastic, isotropic materials based on generalized Hooke's law. 
*   **Behavior:** Stresses are strictly proportional to strains. The material fully recovers its original shape upon unloading.
*   **Parameters:** Young's Modulus $ E $, Poisson's Ratio $ \nu $, Density $ \rho $, and the coefficient of thermal expansion $ \alpha $.

**TOML Example**
```toml
[[materials.linear_elastic]]
id = 1
name = "Standard Concrete"
e = 30000.0
nu = 0.2
rho = 25.0
alpha = 1.0e-5
```

**Python Example**
```python
model.add_linear_elastic_material(
    id=1, 
    name="Standard Concrete", 
    e=30000.0, 
    nu=0.2, 
    rho=25.0, 
    alpha=1.0e-5
)
```

## Bilinear Elasto-Plasticity (von Mises)
A standard non-linear model primarily used for ductile metals like steel. It utilizes the von Mises yield criterion.
* **Behavior:** Linear elastic up to the yield strength $ f_y $. Upon yielding, the material undergoes plastic deformation governed by an isotropic kinematic hardening rule, defined by the tangent modulus $ E_t $.
* **Parameters:** Young's Modulus $ E $, Poisson's Ratio $ \nu $, Yield Strength $ f_y $, Tangent Modulus $ E_t $.

**TOML Example**
```toml
[[materials.elastoplastic]]
id = 2
name = "Steel S235"
e = 210000.0
nu = 0.3
rho = 78.5
fy = 235.0
et = 2100.0   # 1% of E
```

**Python Example**
```python
model.add_elastoplastic_material(
    id=2, 
    name="Steel S235", 
    e=210000.0, 
    nu=0.3, 
    rho=78.5, 
    fy=235.0, 
    et=2100.0
)
```

## Mohr-Coulomb (Geotechnical)
The classic, widely used perfectly-plastic material model for soil and rock mechanics. It evaluates failure based on shear stresses and normal stresses.
* **Behavior:** Defines a hexagonal yield surface in the principal stress space. Once the stress state reaches the yield surface, plastic flow occurs. It incorporates a non-associated flow rule using the dilatancy angle to control volumetric expansion during shearing.
* **Parameters:** Friction Angle ϕ (in degrees), Cohesion c, and Dilatancy Angle ψ (in degrees).

**TOML Example**
```toml
[[materials.mohr_coulomb]]
id = 3
name = "Dense Sand"
e = 50000.0
nu = 0.3
rho = 20.0
phi = 35.0
c = 0.1       # Small cohesion to prevent numerical singularities
psi = 5.0
```

**Python Example**
```python
model.add_mohr_coulomb_material(
    id=3, 
    name="Dense Sand", 
    e=50000.0, 
    nu=0.3, 
    rho=20.0, 
    phi=35.0, 
    c=0.1, 
    psi=5.0
)
```

## Drucker-Prager (Geotechnical / Concrete)
A smooth approximation of the Mohr-Coulomb yield surface, presenting as a cone in the principal stress space.
* **Behavior:** Often used to avoid the numerical singularities (corners) present in the Mohr-Coulomb hexagonal surface. It is highly suitable for massive concrete structures, rock mechanics, and generalized soil modeling.
* **Parameters:** It uses the same engineering inputs (ϕ and c) as Mohr-Coulomb. The engine internally converts these into the Drucker-Prager cone parameters α and k based on the chosen approximation method (e.g., inner or outer cone matching).

**TOML Example**
```toml
[[materials.drucker_prager]]
id = 4
name = "Stiff Clay"
e = 15000.0
nu = 0.35
rho = 19.0
phi = 25.0
c = 15.0
psi = 0.0
```

**Python Example**
```python
model.add_drucker_prager_material(
    id=4, 
    name="Stiff Clay", 
    e=15000.0, 
    nu=0.35, 
    rho=19.0, 
    phi=25.0, 
    c=15.0, 
    psi=0.0
)
```
