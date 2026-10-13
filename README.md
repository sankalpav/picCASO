![picCASO simulation](https://j.gifs.com/6XlqvL.gif)

# picCASO

picCASO is a MATLAB package for simulating how a microbial consortium forms spatial patterns on a 2D surface. Microbial strains in the model communicate through **quorum sensing (QS)**: each strain secretes a signal that boosts the growth of its partner strain. You can use the package to look at how the location of each strain and the strength of the coupling between strains decide the community's final composition and spatial layout.

This repository holds the code and data for the paper:

> **Venkatraghavan S, Anantakrishnan S, Raman K.** Probing patterning in microbial consortia with a cellular automaton for spatial organisation. *Scientific Reports* **12**, 17159 (2022). [doi:10.1038/s41598-022-20705-7](https://doi.org/10.1038/s41598-022-20705-7)

---

## Background

Microbial communities in nature are spatially organised, but they are too complex to study directly. Synthetic consortia are simpler and easier to control. picCASO models a synthetic consortium as a **cellular automaton (CA)** in which each lattice site holds a "cell-block" of bacteria. Each strain carries a growth-reducing gene knockout, and that gene is restored only when the strain senses the signal made by its partner. This ties the growth of each strain to its partners, and the result shows up as spatial patterns.

## Models

### Quorum Sensing–Mediated Model (QSMM) — `QSMM/`
- Each strain constitutively secretes an **acyl homoserine lactone (AHL)**.
- The partner strain's AHL switches on a quorum-sensing promoter that restores the knocked-out gene, which raises the strain's fitness (growth rate).
- Induction works like a Boolean switch: it turns on once the local AHL concentration passes a threshold.
- The paper uses two-strain and three-strain consortia, plus an orthogonal multi-strain design that limits crosstalk between AHL systems.

### Metabolite-Mediated Model (MMM) — `MMM/`
- This is a benchmark model where the strains are **auxotrophs**. Each one needs a metabolite secreted by its partner.
- A strain cannot grow at all without its partner's metabolite (base fitness = 0).

### Fitness from genome-scale modelling
- **Flux balance analysis (FBA)** on the *E. coli* genome-scale model **iAF1260** finds single-gene knockouts that are not lethal but cut growth by roughly 30–90%. These knockouts make up the strain backgrounds.
- You can also supply a genome-scale model for another organism.

### Simulation rules
- 2D grid with a von Neumann (4-neighbour) neighbourhood.
- At every time step, a cell-block divides into an empty neighbouring site with a probability proportional to its fitness.
- AHLs and metabolites diffuse with a Forward-Time Central-Space (FTCS) finite-difference scheme and decay at a first-order rate.
- The main parameters are the **AHL secretion rate**, the **QS activation threshold** and the **gain in fitness** when the knockout is restored.

## Key findings

- **Secretion rate is the most effective control knob.** Changing how fast a strain secretes its AHL reliably shifts the relative populations of the strains, in both two-strain and three-strain consortia. This means you can engineer a target community composition.
- **Thresholds matter only up to a point.** Low activation thresholds favour the induced partner. At high thresholds, AHL levels never reach activation and the populations converge.
- **Fitness gain has little effect** across the range tested.
- **Strains grow toward their partners.** In the preferential-growth assays, strains spread toward their coupled partner across all tested parameter ranges, which shows real QS coupling.
- **The metabolite-mediated model couples more strongly.** It shows about an order of magnitude more growth toward the partner than the QS model. The reason is that MMM strains have zero base fitness, while QSMM strains keep a small non-zero base fitness.
- **Validation against experiments.** The model reproduces the synthetic band-pass-filter ring patterns from [Basu *et al.*, *Nature* 2005](https://doi.org/10.1038/nature03461), and qualitatively reproduces their gallery of patterns for different sender-colony layouts (heart, ellipse, egg and clover). See `Replications/`.

## Repository structure

| Folder | Contents |
|---|---|
| `Package/` | **The main picCASO package.** Run `CoverScript.m`. It includes the FBA step (`FBA.m`), the cell-block class (`cellclass.m`), input handling (`Input.m`) and the built-in seed configurations. |
| `QSMM/` | Quorum sensing–mediated model used in the paper |
| `MMM/` | Metabolite-mediated (auxotroph) benchmark model |
| `Replications/` | Reproductions of the Basu *et al.* (2005) pattern-formation experiments |
| `SensitivityAnalyses/` | Parameter sweeps for `QS_AMT` (secretion rate), `Thresholds` and `GainInFitness`, each for 2- and 3-strain consortia |

## Getting started

### Requirements
- MATLAB (the code was developed on R2019b)
- [COBRA Toolbox](https://opencobra.github.io/cobratoolbox/), needed for the FBA step

### Running a simulation
1. Download the `Package/` folder.
2. Open MATLAB in that folder and run `CoverScript.m`.

### User inputs
- A genome-scale metabolic model (optional; the default is *E. coli* iAF1260 in `model.mat`)
- Selected genes (knockouts)
- Generation time
- Device–AHL pairs (QS sender/receiver modules; defaults are in `DeviceData.mat`)
- A seed configuration. You can pick a built-in one or draw your own in `Blank.m`.

### Built-in seed configurations
| Script | Layout |
|---|---|
| `Lines.m` | Parallel lines |
| `SingleLine.m` | Single file |
| `Square.m` | Diagonally opposite corners |
| `Concentric.m` | Concentric circles |
| `Random.m` | Random distribution |
| `Blank.m` | Custom, user-defined |

### Outputs
Spatial maps of the grid at each time step, and population (cell-block) counts for each strain over time.

## Limitations

- Coupling only reaches short distances, because signal molecules diffuse and degrade. The authors propose positive feedback loops as a way to reach further.
- Gene-expression dynamics inside cells are not modelled, and induction is a Boolean switch.
- The cell-block approximation and the simplified environment (diffusion and degradation only) leave out some biological detail.
- Crosstalk between AHL systems has to be kept down by choosing orthogonal QS pairs.

## Citation

If you use picCASO, please cite:

```bibtex
@article{venkatraghavan2022piccaso,
  title   = {Probing patterning in microbial consortia with a cellular automaton for spatial organisation},
  author  = {Venkatraghavan, Sankalpa and Anantakrishnan, Sathvik and Raman, Karthik},
  journal = {Scientific Reports},
  volume  = {12},
  pages   = {17159},
  year    = {2022},
  doi     = {10.1038/s41598-022-20705-7}
}
```

## Acknowledgements

This work was done at the Indian Institute of Technology Madras (Bhupat and Jyoti Mehta School of Biosciences; Centre for Integrative Biology and Systems mEdicine; Robert Bosch Centre for Data Science and AI). K.R. was funded by SERB MATRICS grant MTR/2020/000490, and S.A. by a DST INSPIRE Scholarship.
