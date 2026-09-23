# Single-Cell RNA-seq Atlas of Healthy Breast Tissue Analysis

A from-scratch reproduction of the single-cell analysis pipeline from 
**Bhat-Nakshatri et al., 2021, *Cell Reports Medicine*** — *"A single-cell atlas of 
the healthy breast tissues reveals clinically relevant clusters of breast epithelial 
cells"* (https://doi.org/10.1016/j.xcrm.2021.100219).

## Biological question

Breast cancer subtypes (luminal A, luminal B, HER2+, basal, claudin-low) are thought 
to arise from distinct cell-of-origin populations within the normal breast epithelium. 
However, the normal breast itself is transcriptionally heterogeneous — composed of 
basal/stem, luminal progenitor, and mature luminal epithelial cells, each with further 
internal subdivisions. Characterizing this normal cellular architecture at single-cell 
resolution is a prerequisite for linking specific epithelial subpopulations to specific 
cancer subtypes, and for identifying markers (such as ER-associated genes) that could 
stratify cancer patients.

This project reproduces the paper's core analysis: profiling healthy donor breast 
tissue at single-cell resolution, identifying the major cell types present, resolving 
fine-grained epithelial subpopulations, and examining a specific finding from the 
paper — that TBX3 and PDK4 are co-expressed with the estrogen receptor gene (ESR1) in 
mature luminal cells, a relationship the original study used to further subclassify 
ER+ breast cancers.

## Data

- **Source:** GEO accession [GSE164898](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE164898)
- **8 samples** from healthy donors: 5 individual samples (D1–D5), 1 individual sample 
  (D11), and 2 pooled cryopreserved sample sets (D6–D10, processed by two independent 
  labs, CMG and HN)
- 10x Genomics Chromium single-cell 3' RNA-seq (`filtered_feature_bc_matrix.h5` per sample)

## Pipeline overview

| Notebook | What it does |
|---|---|
| `01_QC_and_preprocessing.ipynb` | Load 8 samples, compute QC metrics (genes/cell, UMI counts, %mitochondrial), filter low-quality cells, remove doublets (Scrublet), normalize + log-transform, combine into one dataset |
| `02_integration_and_major_celltypes.ipynb` | HVG selection, PCA, Harmony batch integration across samples, Leiden clustering, annotate major cell types (epithelial, T cell, NK cell, myeloid, endothelial, fibroblast) via marker genes |
| `03_epithelial_subclustering_annotation.ipynb` | Subset epithelial cells, re-cluster independently to resolve fine-grained heterogeneity, remove residual non-epithelial contamination, annotate into the paper's three epithelial lineages (basal/stem, luminal progenitor, mature luminal) |
| `04_tbx3_pdk4_analysis.ipynb` | Examine TBX3/PDK4 co-expression with ESR1 in mature luminal cells; statistically test the association (chi-square) |

## Key results

- **45,858** cells passed QC and doublet filtering across all 8 samples
- Batch integration (Harmony) resolved sample-driven clustering artifacts — cluster 
  count dropped from 36 (uncorrected) to 24 (corrected) on the full dataset, with 
  even sample mixing confirmed visually per cluster
- Major cell types identified: epithelial, T cells, NK cells, myeloid, endothelial, 
  fibroblast — consistent with the cell types reported in the paper
- Epithelial cells (13,247) were subset and re-clustered, initially yielding 23 
  clusters (matching the paper's reported epithelial subcluster count); after removing 
  non-epithelial contamination via marker-gene inspection, 19 clean epithelial clusters 
  remained
- These clusters were annotated into the paper's three epithelial lineages — basal/stem, 
  luminal progenitor, mature luminal — using the same marker genes reported in the 
  original study (e.g. CXCL14/ACTA2 for basal, SFRP1/SLPI/KIT/CALML5 for luminal 
  progenitor, PIP/MUCL1/ANKRD30A/XBP1/STC2 for mature luminal)
- TBX3 and PDK4 showed significant positive co-expression in ESR1+ mature luminal 
  cells (chi-square = 47.63, p = 5.14e-12), reproducing the paper's central finding 
  linking these genes to ER+ cell identity

## How to run

1. Download the 8 `.h5` files for GSE164898 from GEO
2. Update file paths in `01_QC_and_preprocessing.ipynb` to match your local setup
3. Run notebooks in order (01 → 04); each notebook saves a checkpoint `.h5ad` file 
   that the next notebook loads

## Environment

- Python 3.10, scanpy, scanpy.external (Harmony via `harmonypy`), pandas, numpy, scipy

## Reference

Bhat-Nakshatri, P., Gao, H., Sheng, L., et al. (2021). A single-cell atlas of the 
healthy breast tissues reveals clinically relevant clusters of breast epithelial 
cells. *Cell Reports Medicine*, 2(3), 100219. https://doi.org/10.1016/j.xcrm.2021.100219
