# FoamAgentCases

A library of **211 validated, runnable OpenFOAM v2412 cases** generated and
verified by the [foamllm3 OpenFOAM AI Agent](https://huggingface.co/datasets/arungovindneelan/openfoam-Agent-Dataset).

Every case in this repository:

- was set up by an LLM-driven CFD agent from a natural-language request,
- was meshed (block-structured or gmsh-converted) to `polyMesh/`,
- was **executed end-to-end with its OpenFOAM solver and converged** within
  our validation pipeline,
- carries a quality score (0 – 1) recorded in `meta.json`.

This is a companion artefact to the
[**openfoam-Agent-Dataset**](https://huggingface.co/datasets/arungovindneelan/openfoam-Agent-Dataset)
on HuggingFace — that dataset is the LLM training corpus
(prompt → file-content pairs), and *this* repository is the **runnable case
bundle** behind those records: clone, change into any case folder, and run.

---

## Quick start

```bash
git clone https://github.com/AGN000/FoamAgentCases.git
cd FoamAgentCases/icoFoam/cavity_re100
./Allrun                 # runs the solver, log written to log.icoFoam
```

Requires **OpenFOAM v2412** (or any release with compatible BC dictionaries).
Source the OpenFOAM environment first:

```bash
source /opt/openfoam2412/etc/bashrc
```

---

## Repository layout

```
FoamAgentCases/
├── README.md
├── LICENSE
├── simpleFoam/               # 105 steady incompressible RANS cases
│   ├── cavity_re100/
│   │   ├── 0/                # initial / boundary conditions
│   │   ├── constant/
│   │   │   ├── polyMesh/     # pre-generated mesh
│   │   │   ├── transportProperties
│   │   │   └── turbulenceProperties
│   │   ├── system/
│   │   │   ├── controlDict
│   │   │   ├── fvSchemes
│   │   │   └── fvSolution
│   │   ├── Allrun            # one-line shell wrapper
│   │   └── meta.json         # solver, original prompt, score, params
│   ├── pipe_re5000/
│   └── ...
├── pimpleFoam/               # 7 transient incompressible cases
├── icoFoam/                  # 39 laminar transient cases
├── buoyantSimpleFoam/        # 20 buoyancy / heat-transfer cases
├── interFoam/                # 18 multiphase (VOF) cases
├── rhoSimpleFoam/            # 17 compressible steady cases
└── rhoPimpleFoam/            # 5 compressible transient cases
```

Cases are grouped by **OpenFOAM solver**. Inside each case directory, the
layout is the standard OpenFOAM tutorial structure (`0/`, `constant/`,
`system/`) plus two extras we added:

- `Allrun` — minimal launch script: `<solver> 2>&1 | tee log.<solver>`
- `meta.json` — provenance and metadata (see below)

### `meta.json` schema

```json
{
  "solver": "icoFoam",
  "prompt": "2D lid-driven cavity Re=100, laminar, icoFoam",
  "score": 0.84,
  "runtime_s": 0.23,
  "params": {
    "geometry_type": "box",
    "is_3d": false,
    "length": 1.0,
    "width": 1.0,
    "reynolds_number": 100,
    "inlet_velocity": 1.0,
    "kinematic_viscosity": 0.01,
    "flow_regime": "laminar",
    "turbulence_model": "laminar",
    "is_transient": true,
    "end_time": 5.0
  }
}
```

`params` is the agent's structured `CFDParams` snapshot — useful if you want
to reproduce the same case with a different mesher / solver, or feed the
parameters into your own pipeline.

---

## Solver coverage

| Solver | Cases | Physics |
|---|---|---|
| `simpleFoam` | 105 | Steady incompressible (RANS: k-ω SST, k-ε, laminar) |
| `icoFoam` | 39 | Transient laminar incompressible |
| `buoyantSimpleFoam` | 20 | Steady buoyancy-driven (Boussinesq) |
| `interFoam` | 18 | Two-phase VOF (dam break, sloshing, wave channel) |
| `rhoSimpleFoam` | 17 | Steady compressible |
| `pimpleFoam` | 7 | Transient incompressible (URANS) |
| `rhoPimpleFoam` | 5 | Transient compressible |
| **Total** | **211** | |

## Geometry coverage

Lid-driven cavity (2D / 3D, Re = 100 – 10 000), Poiseuille and turbulent
pipe flow, channel flow, backward-facing step, flow over a 2D cylinder
(Re = 40 – 1 000), NACA 0012 airfoil at varied angle of attack and Re,
axisymmetric wedge, generic ducts, differentially heated cavity (buoyancy),
dam-break / sloshing / wave-channel (VOF), and compressible nozzle / duct
flows.

---

## How these cases were generated

Each case originates from a **natural-language prompt** in the agent's
[prompt catalogue](https://huggingface.co/datasets/arungovindneelan/openfoam-Agent-Dataset). The agent pipeline:

1. **Parses** the prompt into structured `CFDParams` (geometry, Reynolds
   number, regime, fluid, turbulence model, etc.).
2. **Selects a solver** based on those parameters (`simpleFoam`,
   `pimpleFoam`, `icoFoam`, `buoyantSimpleFoam`, `interFoam`,
   `rhoSimpleFoam`, `rhoPimpleFoam`).
3. **Generates a mesh** — block-structured for primitive shapes, gmsh for
   airfoils / wedges / non-orthogonal geometries — and converts to
   `polyMesh/`.
4. **Writes the case** (controlDict, fvSchemes, fvSolution, transport /
   turbulence / thermophysical properties, BCs in `0/`).
5. **Runs the solver** to a short integration window inside a sandboxed
   working directory.
6. **Scores** the run on residual decay, mass conservation, and BC sanity.
7. Only **converged** runs with **score ≥ 0.5** were promoted into this
   repository.

This is why the corpus is small (~200 cases) but every entry is a verified,
self-contained, runnable OpenFOAM v2412 setup.

---

## Intended use

- Reference setups for OpenFOAM beginners across solver families.
- Held-out evaluation cases for CFD-aware code models.
- Sanity-checking your local OpenFOAM build across many solvers in one repo.
- Companion runnable artefact for fine-tuning runs that use the
  [HuggingFace dataset](https://huggingface.co/datasets/arungovindneelan/openfoam-Agent-Dataset).

## Caveats

- All cases target **OpenFOAM v2412**. Earlier or Foundation-flavour OpenFOAM
  may need minor BC-name patching.
- Meshes are simple primitives (block-mesh / basic gmsh). No industrial CAD
  imports, no AMR, no chimera.
- Validation horizon is short (typically a few seconds of integration);
  long-time stability of every case is not guaranteed.
- `rhoPimpleFoam` (compressible transient) is under-represented — moderate-
  Mach cases are numerically fragile; many candidates failed to converge.

## License

MIT.

