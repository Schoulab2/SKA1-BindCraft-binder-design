# SKA1 tubulin-interface binder design with BindCraft

A reproducible computational workflow for designing de novo protein binders against the **microtubule-binding domain (MTBD) of human SKA1** using **BindCraft**.

The design objective was to target residues that participate directly in the **SKA1–tubulin interaction surface**, with the longer-term aim of generating binders that may competitively interfere with SKA1 binding to microtubules.

> **Status:** computational design only. Binding, affinity, specificity, and functional inhibition remain to be experimentally validated.

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
| Shape complementarity | 0.63 |
| Rosetta dG | -54.62 |
| dSASA | 2132.07 Å² |
| Interface residues | 26 |
| Interface H-bonds | 11 |
| Hotspot RMSD | 1.31 Å |
| Target RMSD | 0.71 Å |
| Relaxed clashes | 0 |

## Workflow

```text
SKA1–α/β-tubulin structural model
            ↓
Identify SKA1 tubulin-contact residues
            ↓
Select hotspot set
Y151 / M152 / R155 / K226 / R236
            ↓
BindCraft 4-stage test
            ↓
Independent AF2 validation fails
            ↓
Switch only the design algorithm to 3-stage
            ↓
ProteinMPNN + independent AF2 validation
            ↓
Rosetta/interface filtering
            ↓
Accepted SKA1 binders
```

## Contents

- [Repository structure](#repository-structure)
- [Software setup](#software-setup)
- [Target definition](#target-definition)
- [Hotspot selection](#hotspot-selection)
- [Initial 4-stage experiment](#initial-4-stage-experiment)
- [Failure analysis](#failure-analysis)
- [Successful 3-stage protocol](#successful-3-stage-protocol)
- [Running the design campaign](#running-the-design-campaign)
- [Monitoring the run](#monitoring-the-run)
- [Accepted designs](#accepted-designs)
- [Lead binder](#lead-binder)
- [Comparing accepted designs](#comparing-accepted-designs)
- [Output plots](#output-plots)
- [Biological interpretation](#biological-interpretation)
- [Next analyses](#next-analyses)
- [Limitations](#limitations)
- [Software and citation](#software-and-citation)

## Repository structure

A compact repository structure is recommended:

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
└── figures/
    └── lead_binder/
```

Large intermediate trajectory directories, model weights, caches, and Conda environments should not be committed.

## Software setup

BindCraft was installed in a separate Conda environment on an NVIDIA RTX A6000 workstation.

The complete installation procedure, dependency fixes, and validation steps are documented in the companion repository:

```text
BindCraft-HIVE-Installation
```

Replace the placeholder below with the final repository URL:

```text
https://github.com/<your-github-username>/BindCraft-HIVE-Installation
```

## Target definition

The SKA1 target was extracted from an AlphaFold3 model containing:

```text
α-tubulin
β-tubulin
SKA1 MTBD
Mg2+
GTP
```

The extracted target corresponds approximately to human SKA1 residues:

```text
141–255
```

and contains:

```text
115 aa
```

The SKA1 target was used as chain:

```text
A
```

Input structure:

```text
ska1_tubulin_bound_141_255.pdb
```

## Hotspot selection

Structural analysis of the SKA1–tubulin model identified SKA1 residues contacting α- or β-tubulin.

Five residues were selected as BindCraft hotspots:

| BindCraft numbering | Full SKA1 residue | Rationale |
|---|---:|---|
| A11 | Y151 | Direct tubulin-contact residue |
| A12 | M152 | Direct tubulin-contact residue |
| A15 | R155 | Direct tubulin-contact residue |
| A86 | K226 | Direct tubulin-contact residue |
| A96 | R236 | Direct tubulin-contact residue |

Hotspot string:

```text
A11,A12,A15,A86,A96
```

These hotspots were chosen to bias design toward the tubulin-facing surface of SKA1 rather than an arbitrary exposed surface.

## Initial 4-stage experiment

The first BindCraft experiment used the default 4-stage multimer protocol.

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

After 20 trajectories:

```text
Trajectories: 20
MPNN designs recorded: 0
Accepted: 0
```

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

The dominant bottleneck was therefore **independent AF2 interface validation**, not ProteinMPNN score or downstream Rosetta filtering.

Promising trajectories often deteriorated during the fourth optimization stage.

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

## Successful 3-stage protocol

Create a 3-stage advanced settings file from the default 4-stage multimer settings:

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

## Running the design campaign

Run BindCraft:

```bash
conda activate BindCraft
cd $HOME/BindCraft

python -u ./bindcraft.py \
  --settings ./settings_target/Track4S_BC1C_groove5_3stage.json \
  --filters ./settings_filters/default_filters.json \
  --advanced ./settings_advanced/Track4S_3stage_multimer.json
```

The 3-stage protocol produced accepted designs, unlike the 4-stage experiment.

At an early checkpoint:

```text
39 trajectories
15 MPNN designs recorded
1 accepted design
```

Further sampling eventually produced three accepted PDBs.

## Monitoring the run

### Count trajectories

```bash
tail -n +2 trajectory_stats.csv | wc -l
```

### Count MPNN designs

```bash
[ -f mpnn_design_stats.csv ] && \
tail -n +2 mpnn_design_stats.csv | wc -l || echo 0
```

### Count accepted PDBs

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

and are closely related ProteinMPNN sequence variants.

The 89-aa design came from a separate trajectory:

```text
s606687
```

and represents the strongest independent solution identified so far.

## Lead binder

Name:

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

Selected metrics:

| Metric | Value |
|---|---:|
| Average pLDDT | 0.92 |
| Average pTM | 0.83 |
| Average i_pTM | 0.79 |
| Average i_pAE | 0.20 |
| Shape complementarity | 0.63 |
| Rosetta dG | -54.62 |
| dSASA | 2132.07 Å² |
| Interface residues | 26 |
| Interface H-bonds | 11 |
| Hotspot RMSD | 1.31 Å |
| Target RMSD | 0.71 Å |
| Binder pLDDT | 0.94 |
| Binder pTM | 0.81 |
| Binder pAE | 0.10 |
| Binder RMSD | 1.10 Å |

## Comparing accepted designs

Use the following script to compare the most useful BindCraft metrics:

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

## Output plots

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

## Biological interpretation

The binder was designed against five residues selected directly from the SKA1 tubulin-contacting surface:

```text
Y151
M152
R155
K226
R236
```

All five correspond to direct tubulin contacts in the SKA1–tubulin structural model used to define the design problem.

The lead binder retained the intended hotspot geometry with an average hotspot RMSD of approximately:

```text
1.31 Å
```

This supports the interpretation that the accepted binder engages the intended SKA1 region.

It does **not**, however, prove that the binder competitively inhibits SKA1–tubulin binding.

## Next analyses

The most important next structural analysis is direct comparison of the accepted SKA1–binder complexes with the original SKA1–α/β-tubulin model.

Recommended analyses:

1. Superpose SKA1 in each BindCraft complex onto tubulin-bound SKA1.
2. Calculate binder–α-tubulin steric overlap.
3. Calculate binder–β-tubulin steric overlap.
4. Identify SKA1 residues contacted by each binder.
5. Compare the binder footprint with the SKA1 tubulin-contact footprint.
6. Quantify the fraction of the tubulin-binding surface occluded.
7. Rank the accepted binders specifically for predicted competition with tubulin.

A subsequent AlphaFold3 calculation containing:

```text
α-tubulin
β-tubulin
SKA1 MTBD
designed binder
```

can be used as a complementary competition test.

## Limitations

An accepted BindCraft design is a computational prediction.

It does not establish:

- experimental binding
- binding affinity
- specificity
- soluble expression
- cellular stability
- tubulin displacement
- functional inhibition of SKA1
- cellular activity

Metrics such as:

```text
i_pTM
i_pAE
Rosetta dG
shape complementarity
```

should not be interpreted as direct experimental affinity measurements.

Experimental validation is required.

Potential validation methods include:

- BLI
- SPR
- MST
- pulldown assays
- co-immunoprecipitation
- microtubule competition assays
- cellular localization
- mitotic phenotyping

## Software and citation

BindCraft:

https://github.com/martinpacesa/BindCraft

ColabDesign:

https://github.com/sokrypton/ColabDesign

This repository documents a project-specific workflow. Please cite the original software repositories and associated publications when using these tools.

## Related repository

BindCraft installation and environment setup are documented separately:

```text
BindCraft-HIVE-Installation
```

Replace the link below after the installation repository has been created:

```text
https://github.com/<your-github-username>/BindCraft-HIVE-Installation
```
