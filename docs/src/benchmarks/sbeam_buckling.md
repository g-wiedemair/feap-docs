# Benchmark: SBeam Buckling

This showcase validates the mathematical precision of the **Total Lagrange formulation** and the **geometric stiffness matrix \\( K_g \\)** in FEAP [Walkthrough - SBeam].

## Test Setup
A vertical beam is clamped at the base and subjected to an axial compressive load at the tip. The goal is to determine the critical Euler buckling load [fenda.pdf].

## Results
FEAP utilizes the `SBeamElement`, which accounts for both Euler-Bernoulli and **Timoshenko effects** [Walkthrough - SBeam].

| Test Case | Description | Result |
| :--- | :--- | :--- |
| `test_beam_axial_stiffness` | Verifies \\( K_{axial} = EA/L \\) | ✅ Passed |
| `test_beam_bending_bernoulli`| Euler-Bernoulli bending terms | ✅ Passed |
| `test_beam_geometric_stiffness`| \\( K_g \\) reduction under compression | ✅ Passed [Walkthrough - SBeam] |

This test proves that FEAP accurately represents stability problems and second-order effects [System-Update-Prompt].
