# SKA1 tubulin-interface binder design with BindCraft

De novo design of protein binders targeting the **microtubule-binding domain (MTBD) of human SKA1** using **BindCraft**.

The design campaign was focused specifically on the **SKA1–tubulin interaction surface**. Five SKA1 residues that contact tubulin in the starting structural model were used as BindCraft hotspots, with the aim of generating binders that may sterically compete with SKA1–microtubule binding.

> **Status:** computational design only. Binding, affinity, specificity, and inhibition of SKA1–microtubule binding remain to be tested experimentally.

---

## Table of Contents

- [Key result](#key-result)
- [Design strategy](#design-strategy)
- [Repository structure](#repository-structure)
- [Software setup](#software-setup)
- [Target preparation](#target-preparation)
- [Hotspot selection](#hotspot-selection)
- [Initial 4-stage BindCraft run](#initial-4-stage-bindcraft-run)
- [Failure analysis](#failure-analysis)
- [Successful 3-stage protocol](#successful-3-stage-protocol)
- [Running the successful design campaign](#running-the-successful-design-campaign)
- [Monitoring a BindCraft run](#monitoring-a-bindcraft-run)
- [Accepted designs](#accepted-designs)
- [Lead binder](#lead-binder)
- [Comparing accepted designs](#comparing-accepted-designs)
- [BindCraft output plots](#bindcraft-output-plots)
- [Biological interpretation](#biological-interpretation)
- [Next analyses](#next-analyses)
- [Limitations](#limitations)
- [Related repository](#related-repository)
- [Software and citation](#software-and-citation)

---

## Key result

The current lead candidate is:

```text
SKA1_BC1C_l89_s606687_mpnn11
```

Sequence:

```text
AEIPEKERKEMLEWINWLIEDYAAQYPDKIDVEAEKKEAEKLIEELIKEYTEKFNEGKVDFKTNDDLIYEIGIDADYLVDEKLRQKADA
```

Length:

```text
89 aa
```

Selected BindCraft metrics:

| Metric | Value |
|---|---:|
| Average pLDDT | 0.92 |
| Average pTM | 0.83 |
| Average i_pTM | 0.79 |
| Average pAE | 0.15 |
| Average i_pAE | 0.20 |
| Binder pLDDT | 0.94 |
| Binder pTM | 0.81 |
| Binder pAE | 0.10 |
| Binder RMSD | 1.10 Å |
| Shape complementarity | 0.63 |
| Rosetta dG | -54.62 |
| dSASA | 2132.07 Å² |
| Interface residues | 26 |
| Interface H-bonds | 11 |
| Hotspot RMSD | 1.31 Å |
| Target RMSD | 0.71 Å |
| Relaxed clashes | 0 |

These values are computational quality and interface metrics and should not be interpreted as experimental affinity measurements.

---

## Design strategy

```text
SKA1–α/β-tubulin structural model
            ↓
Identify SKA1 residues contacting tubulin
            ↓
Select hotspot set
Y151 / M152 / R155 / K226 / R236
            ↓
BindCraft 4-stage design
            ↓
Independent AF2 validation fails
            ↓
Change only design algorithm:
4-stage → 3-stage
            ↓
ProteinMPNN + independent AF2 validation
            ↓
Rosetta/interface filtering
            ↓
Accepted SKA1 binders
```

Earlier RFdiffusion-based SKA1 campaigns generated candidate backbones near the target surface but failed downstream sequence/cofold validation. This motivated the switch to BindCraft, which performs integrated optimization of sequence, backbone, and target interface.

---

## Repository structure

Recommended repository layout:

```text
SKA1-BindCraft-binder-design/
├── README.md
├── settings/
│   ├── Track4S_BC1C_groove5_3stage.json
│   └── Track4S_3stage_multimer.json
├── scripts/
│   ├── compare_accepted_designs.py
│   └── monitor_bindcraft_run.sh
├── target/
│   └── ska1_tubulin_bound_141_255.pdb
├── results/
│   ├── lead_sequence.fasta
│   ├── final_design_stats.csv
│   └── accepted_structures/
│       ├── SKA1_BC1C_l89_s606687_mpnn11_model2.pdb
│       ├── SKA1_BC1C_l95_s464187_mpnn5_model2.pdb
│       └── SKA1_BC1C_l95_s464187_mpnn7_model2.pdb
└── figures/
    └── lead_binder/
```

Large trajectory directories, AlphaFold model weights, caches, Conda environments, and other generated intermediates should not be committed to the repository.

---

## Software setup

BindCraft was run in a dedicated Conda environment on an NVIDIA RTX A6000 workstation.

The full installation procedure, dependency fixes, and environment-validation steps are documented separately:

[BindCraft-Installation](https://github.com/Schoulab2/BindCraft-Installation)

---

## Target preparation

The design target was derived from an AlphaFold3 model containing:

```text
α-tubulin
β-tubulin
SKA1 MTBD
Mg2+
GTP
```

The extracted SKA1 target corresponds approximately to human SKA1 residues:

```text
141–255
```

and contains:

```text
115 aa
```

For BindCraft, the target was stored as:

```text
ska1_tubulin_bound_141_255.pdb
```

with SKA1 assigned to:

```text
chain A
```

---

## Hotspot selection

Five SKA1 residues were selected from the tubulin-contacting surface:

| BindCraft numbering | Full SKA1 residue | Rationale |
|---|---:|---|
| A11 | Y151 | Direct tubulin-contact residue |
| A12 | M152 | Direct tubulin-contact residue |
| A15 | R155 | Direct tubulin-contact residue |
| A86 | K226 | Direct tubulin-contact residue |
| A96 | R236 | Direct tubulin-contact residue |

The hotspot string used by BindCraft was:

```text
A11,A12,A15,A86,A96
```

The intention was to bias design toward the **tubulin-facing surface of SKA1**, rather than an arbitrary solvent-exposed site.

---

## Initial 4-stage BindCraft run

The initial BindCraft experiment used the default 4-stage multimer protocol.

Target settings:

```json
{
  "binder_name": "SKA1_BC1A",
  "chains": "A",
  "target_hotspot_residues": "A11,A12,A15,A86,A96",
  "lengths": [70, 120],
  "number_of_final_designs": 20
}
```

Run:

```bash
conda activate BindCraft
cd $HOME/BindCraft

python -u ./bindcraft.py \
  --settings ./settings_target/Track4S_BC1A_groove5.json \
  --filters ./settings_filters/default_filters.json \
  --advanced ./settings_advanced/default_4stage_multimer.json
```

After 20 completed trajectories:

```text
Trajectories: 20
MPNN designs recorded: 0
Accepted: 0
```

---

## Failure analysis

The largest failure categories were:

```text
i_pAE                             398
i_pTM                             394
pTM                               266
pLDDT                              96
Trajectory_Clashes                 18
Trajectory_logits_pLDDT             6
Trajectory_final_pLDDT              5
Trajectory_one-hot_pLDDT            4
Trajectory_softmax_pLDDT            3
MPNN_score                          0
MPNN_seq_recovery                   0
```

The dominant bottleneck was therefore **independent AF2 interface validation**, rather than ProteinMPNN score or downstream Rosetta filtering.

Promising trajectories were frequently observed before the final stage but then deteriorated during the fourth optimization stage.

The next experiment therefore kept the same:

```text
target
hotspots
binder-length range
filters
```

and changed only:

```text
design_algorithm: 4stage → 3stage
```

---

## Successful 3-stage protocol

A 3-stage configuration was created from the default 4-stage multimer settings by changing only `design_algorithm`.

```bash
cd $HOME/BindCraft

python - <<'PY'
import json

src = "settings_advanced/default_4stage_multimer.json"
dst = "settings_advanced/Track4S_3stage_multimer.json"

with open(src) as f:
    x = json.load(f)

x["design_algorithm"] = "3stage"

with open(dst, "w") as f:
    json.dump(x, f, indent=2)

print("Created:", dst)
print("design_algorithm =", x["design_algorithm"])
print("soft_iterations =", x["soft_iterations"])
print("temporary_iterations =", x["temporary_iterations"])
print("hard_iterations =", x["hard_iterations"])
PY
```

Successful target settings:

```json
{
  "binder_name": "SKA1_BC1C",
  "chains": "A",
  "target_hotspot_residues": "A11,A12,A15,A86,A96",
  "lengths": [70, 120],
  "number_of_final_designs": 20
}
```

---

## Running the successful design campaign

```bash
conda activate BindCraft
cd $HOME/BindCraft

python -u ./bindcraft.py \
  --settings ./settings_target/Track4S_BC1C_groove5_3stage.json \
  --filters ./settings_filters/default_filters.json \
  --advanced ./settings_advanced/Track4S_3stage_multimer.json
```

The 3-stage protocol generated accepted designs.

At an early checkpoint:

```text
39 trajectories
15 MPNN designs recorded
1 accepted design
```

Continued sampling produced three accepted PDB structures.

---

## Monitoring a BindCraft run

### Count completed trajectories

```bash
tail -n +2 trajectory_stats.csv | wc -l
```

### Count MPNN designs recorded

```bash
[ -f mpnn_design_stats.csv ] && \
tail -n +2 mpnn_design_stats.csv | wc -l || echo 0
```

### Count accepted structures

```bash
find Accepted -maxdepth 1 -type f -name "*.pdb" | wc -l
```

### List accepted structures

```bash
find Accepted \
  -maxdepth 1 \
  -type f \
  -name "*.pdb" \
  -printf "%f\n"
```

### Inspect failure statistics

```bash
python - <<'PY'
import pandas as pd

df = pd.read_csv("failure_csv.csv")
print(df.iloc[0].sort_values(ascending=False).to_string())
PY
```

---

## Accepted designs

Three designs were accepted:

```text
SKA1_BC1C_l89_s606687_mpnn11_model2.pdb
SKA1_BC1C_l95_s464187_mpnn5_model2.pdb
SKA1_BC1C_l95_s464187_mpnn7_model2.pdb
```

The two 95-aa designs share the same seed/backbone:

```text
s464187
```

and are therefore closely related ProteinMPNN sequence variants.

The 89-aa design was generated from a separate trajectory:

```text
s606687
```

and represents an independent structural solution.

---

## Lead binder

The strongest candidate based on the combined BindCraft metrics was:

```text
SKA1_BC1C_l89_s606687_mpnn11
```

Sequence:

```text
AEIPEKERKEMLEWINWLIEDYAAQYPDKIDVEAEKKEAEKLIEELIKEYTEKFNEGKVDFKTNDDLIYEIGIDADYLVDEKLRQKADA
```

Length:

```text
89 aa
```

Accepted structure:

```text
SKA1_BC1C_l89_s606687_mpnn11_model2.pdb
```

Compared with the two 95-aa accepted variants, the 89-aa design showed:

- higher `i_pTM`
- lower `i_pAE`
- larger buried interface area
- more interface residues
- more interface hydrogen bonds
- more favorable Rosetta `dG`
- lower hotspot RMSD

The 95-aa designs showed somewhat higher shape complementarity but were weaker overall by the combined interface-confidence metrics.

---

## Comparing accepted designs

The following script extracts the principal metrics from `final_design_stats.csv`:

```python
import pandas as pd

df = pd.read_csv("final_design_stats.csv")

cols = [
    "Design",
    "Length",
    "Seed",
    "Sequence",
    "MPNN_score",
    "MPNN_seq_recovery",
    "Average_pLDDT",
    "Average_pTM",
    "Average_i_pTM",
    "Average_pAE",
    "Average_i_pAE",
    "Average_ShapeComplementarity",
    "Average_PackStat",
    "Average_dG",
    "Average_dSASA",
    "Average_dG/dSASA",
    "Average_n_InterfaceResidues",
    "Average_n_InterfaceHbonds",
    "Average_n_InterfaceUnsatHbonds",
    "Average_Hotspot_RMSD",
    "Average_Target_RMSD",
    "Average_Binder_pLDDT",
    "Average_Binder_pTM",
    "Average_Binder_pAE",
    "Average_Binder_RMSD"
]

print(df[cols].to_string(index=False))
```

Current computational ranking:

```text
1. SKA1_BC1C_l89_s606687_mpnn11
2. SKA1_BC1C_l95_s464187_mpnn7
3. SKA1_BC1C_l95_s464187_mpnn5
```

---

## BindCraft output plots

For the 89-aa lead binder, BindCraft generated:

```text
SKA1_BC1C_l89_s606687_plddt.png
SKA1_BC1C_l89_s606687_ptm.png
SKA1_BC1C_l89_s606687_i_ptm.png
SKA1_BC1C_l89_s606687_pae.png
SKA1_BC1C_l89_s606687_i_pae.png
SKA1_BC1C_l89_s606687_loss.png
SKA1_BC1C_l89_s606687_con.png
SKA1_BC1C_l89_s606687_i_con.png
SKA1_BC1C_l89_s606687_rg.png
```

BindCraft also generated an optimization animation:

```text
SKA1_BC1C_l89_s606687.html
```

These plots describe the optimization trajectory. Final accepted-design statistics should be taken from `final_design_stats.csv`.

---

## Biological interpretation

The lead binder was explicitly designed against five SKA1 residues selected from the tubulin-contacting surface:

```text
Y151
M152
R155
K226
R236
```

In the SKA1–tubulin model used to define the design problem, all five residues participate directly in tubulin contacts.

The lead candidate retained the intended hotspot geometry with an average hotspot RMSD of approximately:

```text
1.31 Å
```

This supports the interpretation that the design engages the intended region of SKA1.

It does **not** establish that the binder can displace tubulin or inhibit SKA1 function experimentally.

---

## Next analyses

The most direct next structural analysis is to compare the accepted SKA1–binder complexes with the original SKA1–α/β-tubulin model.

Recommended analyses:

1. Superpose SKA1 from each BindCraft complex onto tubulin-bound SKA1.
2. Calculate binder–α-tubulin steric overlap.
3. Calculate binder–β-tubulin steric overlap.
4. Identify SKA1 residues contacted by each binder.
5. Compare the binder footprint with the SKA1 tubulin-contact footprint.
6. Quantify how much of the tubulin-binding surface is occluded.
7. Rank accepted designs specifically for predicted competition with tubulin.

A complementary AlphaFold3 test can include:

```text
α-tubulin
β-tubulin
SKA1 MTBD
designed binder
```

This should be treated as an additional structural prediction rather than direct evidence of binding competition.

---

## Limitations

An accepted BindCraft design is a **computational prediction**.

It does not establish:

- experimental binding
- quantitative binding affinity
- target specificity
- soluble expression
- cellular stability
- tubulin displacement
- inhibition of SKA1 function
- cellular activity

Metrics including:

```text
i_pTM
i_pAE
Rosetta dG
shape complementarity
```

are useful for computational ranking but are not experimental affinity measurements.

Potential experimental validation methods include:

- BLI
- SPR
- MST
- pulldown assays
- co-immunoprecipitation
- microtubule competition assays
- cellular localization
- mitotic phenotyping

---

## Related repository

Detailed BindCraft installation and environment setup:

[BindCraft-Installation](https://github.com/Schoulab2/BindCraft-Installation)

---

## Software and citation

BindCraft:

https://github.com/martinpacesa/BindCraft

ColabDesign:

https://github.com/sokrypton/ColabDesign

This repository documents a project-specific workflow. Please consult and cite the official software repositories and associated publications when using these tools in scientific work.
