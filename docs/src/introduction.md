# Introduction

Welcome to the documentation for **FEAP** (FENDA-Rust Port & Editor).

FEAP is a modern, high-performance finite element software written entirely in the Rust systems programming language. The project bridges the gap between classical, proven engineering mechanics and modern software concepts such as extreme memory safety, aggressive multi-threading, and interactive GPU visualization.

## Historical Reference and Theoretical Basis

The mathematical and physical foundations of this project are largely based on the established FEM system **FENDA** [1]. The implementation of the continuum elements and the formulation of the underlying variational principles (e.g., the Rayleigh-Ritz principle) are closely oriented towards the theoretical foundations of FENDA. 

The goal of FEAP is to transfer these proven mechanical concepts into a modern software architecture, making calculations faster, safer, and more usable within automated processes (scripting).

## Core Concepts of the Engine

Over the course of development, FEAP has evolved from a simple linear truss solver into a comprehensive numerical platform. The most important core concepts include:

*   **Extensive Element Library:** The engine supports 3D trusses (including large deformations and thermal expansion \\(\Delta T\\)), exact spatial beam kinematics (SBeam with geometric stiffness \\(K_g\\) according to Timoshenko), as well as 2D continuum elements (`Quad4`, `Quad8`) for geotechnical engineering and planar structures.
*   **Non-Linear Sequencing & Construction Stages:** The integrated Newton-Raphson solver supports complex load paths through **full physical state inheritance** (`continue_from`). Nodal displacements, internal stresses, and the plastic history of integration points can be seamlessly transferred into new construction stages.
*   **Total-Load Approach:** Instead of having to manually calculate load increments for non-linear phases, the user in FEAP always defines the absolute target state (the total load) of a phase. The solver automatically and smoothly interpolates external forces from the inherited starting state to the new target over the specified load steps.
*   **Scripting & Automation:** In addition to the interactive 3D user interface (wgpu/Iced), FEAP offers deep **Python integration**. Models can be built entirely via scripts, iteratively solved, and evaluated, making FEAP an ideal tool for parametric studies, ultimate load determination, and optimization loops.

## Target Audience

This manual is intended for structural engineers, geotechnical engineers, and researchers who require a robust and physically precise tool for non-linear structural and continuum analyses, without wanting to sacrifice the convenience of a modern scripting language and a responsive user interface.
