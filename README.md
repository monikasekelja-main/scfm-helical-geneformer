# Gene Knockout Analysis with GeneFormer V2 Embeddings

**In-silico perturbation of ALS-associated genes in motor cortex snRNA-seq data**


---

## Overview

This project applies GeneFormer V2 (via the [Helical](https://github.com/helicalAI/helical) Python package) to simulate in-silico gene perturbations in ALS-vulnerable motor cortex neurons and interpret the resulting embedding shifts for drug target prioritisation.

The analysis uses the Pineda et al. (*Cell* 2024) motor cortex snRNA-seq dataset (Brodmann Area 4, primary motor cortex) containing 112,014 cells from sporadic ALS and pathologically normal donors. A curated 10-gene panel (see slides/ALS_Priority_Gene_Targets.pdf) spanning five mechanistic ALS axes is systematically perturbed in two disease-vulnerable populations (VAT1L⁺ upper motor neurons and SCN4B⁺ deep-layer projection neurons) using a paired-cell experimental design.

### Key Findings

- **STMN2** is the top-ranked drug target candidate (composite score 0.776), with bidirectional consistency, cross-population replication, FDR significance, and a monotonic dose-response (R² = 0.987)
- The ALS gene panel shows collective functional distinctiveness vs random genes (Mann-Whitney p = 2.2 × 10⁻⁶⁸), despite individual genes not passing FDR correction in disease cells
- ACTB negative control validates framework specificity (near-zero directional shift across all populations)
- Pretrained GeneFormer V2 places sALS and healthy motor neurons at cosine distance 0.003–0.006, constraining individual effect sizes to ~10⁻⁴ — fine-tuning would amplify signals by 10–100×

## Repository Structure

```
scfm-coding-challenge/
├── README.md
├── notebooks/
│   ├── 01_perturbation_workflow.ipynb    # Task 1: Generic perturbation framework
│   ├── 02_als_gene_perturbations.ipynb   # Task 2: ALS gene panel perturbations
│   ├── 03_embedding_interpretation.ipynb # Task 3: Embedding space interpretation
│   └── 04_drug_target_prioritisation.ipynb # Task 4: Drug target ranking
├── data/                                 # Dataset (downloaded separately)
├── results/                              # Exported embeddings, CSVs, pickles
├── figures/                              # All generated figures
└── slides/                               # Summary slide deck and information on ALS gene panel selection
```

## Notebooks

| Notebook | Task | Description |
|----------|------|-------------|
| **01** | Perturbation Workflow | Reusable `InSilicoPerturbation` class: knockdown, knockup, dose-response, null distributions, paired-cell design |
| **02** | ALS Gene Perturbations | 10-gene × 4-population systematic perturbation (44 experiments), FDR correction, dose-response, combinatorial perturbations |
| **03** | Embedding Interpretation | UMAP, PCA trajectory decomposition, k-NN neighbourhood analysis, hierarchical clustering, bidirectional consistency |
| **04** | Drug Target Prioritisation | 6-metric composite scoring, radar charts, null validation, dose-response modelling, fine-tuning projections, final ranking |

## Dataset

The curated dataset is provided by Helical AI (pre-signed S3 URL in the challenge instructions). It contains snRNA-seq count data from Brodmann Area 4 (primary motor cortex) for sporadic ALS and pathologically normal donors, derived from the Pineda et al. (*Cell* 2024) atlas.

**Source publication:** Pineda et al., "Single-cell dissection of the human motor and prefrontal cortices in ALS and FTLD", *Cell* 187, 1971–1989 (2024).

---

## Setup: AWS GPU Instance

### 1. Launch the EC2 instance

In the AWS Console (recommended region: `eu-west-2` London):

- **AMI:** `Deep Learning OSS Nvidia Driver AMI GPU PyTorch 2.10 (Ubuntu 24.04)`
- **Instance type:** `g5.xlarge` (NVIDIA A10G, 24 GB VRAM, 4 vCPUs, 16 GB RAM)
- **Storage:** 100 GB gp3 (the dataset + model weights need space)
- **Key pair:** Select or create a `.pem` key
- **Security group:** Allow SSH (port 22) from your IP

### 2. Configure SSH on your local machine

Add the instance to `~/.ssh/config`:

```
Host helical-gpu
    HostName <Public IPv4 address of your AWS instance>
    User ubuntu
    IdentityFile ~/.ssh/<your-key>.pem
```

Then connect:

```bash
ssh helical-gpu
```

### 3. Install Conda on AWS instance

The Deep Learning AMI comes with system Python and CUDA drivers, but we need a clean conda environment:

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh
bash ~/miniconda.sh -b -p ~/miniconda3
rm ~/miniconda.sh

~/miniconda3/bin/conda init bash
source ~/.bashrc
```

### Create the environment and install dependencies

```bash
conda create -n helical python=3.10 -y
conda activate helical
```

```bash
pip install helical scanpy numpy scipy pandas matplotlib seaborn scikit-learn torch umap-learn statsmodels
```


```bash
pip install ipykernel
python -m ipykernel install --user --name helical --display-name "helical"
```

### 4. Clone the repository

```bash
git clone https://github.com/monikasekelja-main/scfm-coding-challenge.git
```


### 5. Download the dataset

Ask the Helical AI team to provide you with the S3 AWS URL to counts_combined_filtered_BA4_sALS_PN.h5ad and store it in the /data directory


--

## Technical Notes

### Model versioning

The challenge specifies `gf-12L-95M-i4096`. The Helical API automatically redirects this to `gf-12L-38M-i4096` with a deprecation warning — they are the same model. Both names are documented in the notebooks for transparency.

### GPU memory

- Embedding (inference): fits comfortably on g5.xlarge (24 GB VRAM)
- Fine-tuning: exceeds 24 GB for the 12-layer model even with 10 frozen layers and 300 cells. Requires g5.2xlarge (48 GB) or bigger (haven't tested) ...

### Perturbation design

The perturbation framework modifies the raw count matrix and re-processes through the Helical API's `process_data()` → `get_embeddings()` pipeline. This is functionally equivalent to GeneFormer's native in-silico deletion/activation (operating on rank value encodings) since setting counts to zero removes the gene from the encoding, and setting counts high moves it to the top rank.

---

## References

1. Theodoris CV et al. "Transfer learning enables predictions in network biology." *Nature* 618, 616–624 (2023).
2. Pineda SS et al. "Single-cell dissection of the human motor and prefrontal cortices in ALS and FTLD." *Cell* 187, 1971–1989 (2024).
3. Helical AI. *helical* Python package. https://github.com/helicalAI/helical

---

## License

This repository contains code developed for the Helical AI coding challenge. The dataset is provided under the terms specified by Helical AI and the original data generators (GEO: GSE174332).
