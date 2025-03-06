# Comprehensive VASP Guide for V-O System Calculations

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [System Requirements](#system-requirements)
3. [Directory Structure](#directory-structure)
4. [Input File Preparation](#input-file-preparation)
5. [Formation Energy Calculations](#formation-energy-calculations)
6. [Electronic Structure Calculations](#electronic-structure-calculations)
7. [Troubleshooting](#troubleshooting)
8. [Post-processing](#post-processing)
9. [References](#references)

## Prerequisites

### Software Requirements
- VASP 5.4.4 or newer (tested with 5.4.4 and 6.1.0)
- Optional but recommended:
  - VASPKIT (version 1.2.1+)
  - Pymatgen (version 2022.0.0+)

### Required Skills
- Basic Linux command line operations
- Understanding of DFT concepts
- Familiarity with crystal structures

## System Requirements

### Computational Resources
- Minimum RAM: 8GB per core
- Recommended cores: 16-64 depending on system size
- Storage: ~15GB (depends on your calculation)
- GPU support: Optional but beneficial for VASP 6+

### Storage Organization
   ```
   formation_energy/
   ├── Stable_VO2/
       ├── INCAR
       ├── KPOINTS
       ├── POTCAR
       └── .vasp files (POSCAR)
   ├── Stable_V2O3/
       ├── INCAR
       ├── KPOINTS
       ├── POTCAR
       └── .vasp files (POSCAR)
   ├── Stable_V2O5
       ├── INCAR
       ├── KPOINTS
       ├── POTCAR
       └── .vasp files (POSCAR)
   └─── DOS and Band Structure Calculation
       └── .vasp files (POSCAR)
   ```

## Input File Preparation

### POSCAR Validation
- Structure file format example (for VO2):
```
V O
1.0
    6.3887779019977433    0.0000000000000000    0.0000000000000000
   -1.6077386455938496    5.2103739898368104    0.0000000000000000
   -0.1798293348652883    0.5121936143580871    4.3647284837355809
1 2
Cartesian
  4.2537299607693022  3.3300338020974487 -0.1613857581322096
 -0.4337700392306978  1.9237838020974487  0.0729892418677904
  3.0818549607693022  3.3300338020974487  6.6354892418677904
```

Validation checklist:
- Correct atomic ordering (match POTCAR sequence)
- No overlapping atoms (use VESTA or similar for visualization)
- Consistent units (Å for distances)

### POTCAR Selection
- V: PAW PBE V_pv (recommended for oxide calculations)
- O: PAW PBE O
- Generate POTCAR using:
```bash
cat $VASP_PP_PATH/V_pv/POTCAR $VASP_PP_PATH/O/POTCAR > POTCAR
```

### KPOINTS Generation
- For structure optimization:
```
Automatic mesh
0
Gamma
4 4 4
0 0 0
```


## Formation Energy Calculations

### INCAR Settings
```
# Base Parameters
SYSTEM = V-O_formation
PREC = Accurate
ENCUT = 500
ISPIN = 2
ALGO = Fast
NELM = 100

# Electronic Parameters
EDIFF = 1E-6
ISMEAR = -5    # For semiconductors/insulators
SIGMA = 0.05

# Ionic Parameters
IBRION = 2
ISIF = 3       # Full cell optimization
NSW = 100
EDIFFG = -0.01

# DFT+U Parameters
LDAU = .TRUE.
LDAUTYPE = 2
LDAUL = 2 -1   # U for V d-orbitals
LDAUU = 3.25 0.0
LDAUJ = 0.0 0.0
```
### Run Calculations
   ```bash
       cd $dir
       mpirun -np X vasp_std > vasp.out
   ```

## Electronic Structure Calculations

### DOS Calculation Workflow
1. Structure relaxation (as above)
2. Static SCF calculation
3. DOS calculation
4. Analysis

Updated INCAR for DOS:
```
# Previous DFT+U parameters remain same
ISMEAR = -5
SIGMA = 0.02
NEDOS = 2000
LORBIT = 11
ICHARGE = 11
```

### Band Structure Workflow
1. SCF calculation
2. Generate k-path
3. Non-SCF calculation along k-path
4. Analysis

K-path generation using VASPKIT:
```bash
vaspkit -task 303
```

## Troubleshooting

### Common Issues and Solutions

1. SCF Convergence Problems
   - Increase NELM
   - Try different ALGO settings
   - Adjust mixing parameters (AMIX, BMIX)

2. Geometry Optimization Issues
   - Check initial structure
   - Adjust EDIFFG
   - Monitor forces and stress tensor

### Validation Checks
1. Energy convergence
2. Forces < 0.01 eV/Å
3. Stress tensor components < 1 kBar
4. Magnetic moments physical
5. Band gap reasonable (for semiconductors)

## Post-processing

### Formation Energy Analysis
```python
def calculate_formation_energy(E_compound, E_V, E_O2, n_V, n_O):
    """
    Calculate formation energy per atom
    """
    E_form = (E_compound - (n_V * E_V + n_O * E_O2/2)) / (n_V + n_O)
    return E_form
```

### DOS Analysis using VASPKIT
```bash
vaspkit -task 111  # Total DOS
```

### Band Structure Plotting
```python
from pymatgen.io.vasp import BSVasprun
from pymatgen.electronic_structure.plotter import BSPlotter

vasprun = BSVasprun("vasprun.xml")
bs = vasprun.get_band_structure()
plotter = BSPlotter(bs)
plotter.save_plot("bands.png")
```

## References

1. VASP Manual: [https://www.vasp.at/wiki/index.php/The_VASP_Manual](https://www.vasp.at/wiki/index.php/The_VASP_Manual)
2. U-J Parameters: Physical Review B XX, XXXXX (YEAR)
3. Methodology: Computational Materials Science XX, XXXXX (YEAR)

## Notes
- Always check convergence with respect to k-points and ENCUT
- Backup calculations regularly
- Monitor job progress using tail -f vasp.out
- Consider symmetry implications when using ISIF=3
- Document any deviations from these settings

---
Last updated: December 2024
