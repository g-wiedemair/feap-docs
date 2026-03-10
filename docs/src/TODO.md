# TODO

* Features:
    -  9. Python hot reloading
    
    - 13: Geotechnik
        - Construction Stages
        - GAP Element
        - Materialmodelle: Drucker-Prager, Mohr-Coulomb, Hardening Soil
        - Rotationssymmetrie
        - Als Beispiel das Wasserspeicherproblem (Erddruck oben erhöht aktiv, unten SiloDruck)
        - 
        - export Truss/Beam/Plate to FENDA and verify
        - Ergebnisbilder exportieren
        - Wie können wir eine Abfolge von Ergebnissen speichern, python?
        - Stabzug 2D mit mehreren Kräften darstellen (Trace Line)
        - Visibility Labels/Axes/TraceLine
        - Templates
        
* Materialien:
    - Nichtlineare Materialien (Beyond BiLinear Steel)
        - Mohr-Coulomb
        - Drucker-Prager
        
* Elements:
    - Beam Element
        - Vorspannung
        - Wölbkrafttorsion
        - Vergleich mit Biegedrillknicken nach EC
    - Gap Element
    - Plane Strain
        - Berechnung Erddruck

* PERFORMANCE: 
    - Lookup node -> connected elements
    - bounding box für raycasting
    - GPU-Beschleunigung:
        - Mesh Shader: komplexes Geometrie-Processing und Culling
        - GPU-Assembling