# rare-variant-exomechip-gwas-qc

## Pre-processing Rare-Variant GWAS Exome-Chip Data Using Illumina GenomeStudio and PLINK

## Overview

This repository describes a standard preprocessing workflow for **rare-variant SNP exome-chip datasets** using **Illumina GenomeStudio** for genotype calling and cluster inspection, followed by **PLINK** for file conversion and quality control.

Exome-chip datasets differ from conventional common-variant GWAS arrays because they are enriched for **rare coding variants**, which are often harder to call accurately with fully automated clustering. As a result, preprocessing must include both:

- **GenomeStudio-level genotype calling and cluster review**
- **PLINK-based sample and SNP quality control**

The final goal is to generate a **clean binary PLINK dataset** (`.bed/.bim/.fam`) suitable for downstream analyses such as:

- rare-variant GWAS
- burden testing
- gene-based association analysis
- population structure and relatedness checks

---

## Objectives

This workflow is designed to:

- import and genotype Illumina exome-chip data in GenomeStudio
- inspect and improve rare-variant cluster calls
- export genotype reports for downstream processing
- convert data into PLINK format
- perform sample-level and variant-level QC
- produce a reproducible cleaned dataset for association testing

---

## Input Data

Typical input files include:

- Illumina intensity and genotype data loaded into **GenomeStudio**
- array manifest files
- cluster definition files (`.egt`), if available
- sample sheet / phenotype sheet
- metadata such as:
  - sample IDs
  - sex
  - phenotype
  - plate or batch
  - ancestry labels
  - case/control status

---

## Software Requirements

- **Illumina GenomeStudio Genotyping Module**
- **PLINK** (1.9 or 2.0, depending on downstream needs)
- Optional:
  - R / Python for QC visualization
  - zCall or equivalent rare-variant recall tools
  - ancestry reference panels for PCA

---

## Recommended Workflow

## 1. Import raw data into GenomeStudio

Begin by creating or opening a GenomeStudio project and loading the appropriate:

- manifest file
- sample sheet
- intensity data
- cluster file (`.egt`), if one exists for the array or population

At import time, enable options to:

- calculate sample statistics
- calculate SNP statistics
- cluster SNPs
- generate genotype calls

This stage produces the initial genotype calls and marker-level quality summaries.

---

## 2. Perform initial SNP and sample review in GenomeStudio

After genotype calling, inspect standard GenomeStudio QC metrics, especially:

- **Call Frequency**
- **GenTrain Score**
- **Cluster Separation**

These are especially important for exome-chip studies because rare variants often form weak or sparse genotype clusters. Automated calling may incorrectly label rare heterozygotes, collapse clusters, or mark true rare variants as missing.

At this step, flag SNPs that show:

- poor clustering
- low call frequency
- low GenTrain score
- unusual cluster shape
- batch-specific artifacts

---

## 3. Manually inspect and refine rare-variant clusters

Rare variants often require manual review. In GenomeStudio, inspect cluster plots for SNPs that are:

- biologically important
- associated with known candidate genes
- rare but potentially real
- poorly called by the default clustering algorithm

Typical actions include:

- reclustering SNPs
- adjusting cluster boundaries
- confirming rare heterozygote calls
- excluding markers with irreparable signal patterns

This is one of the most important preprocessing steps for exome-chip data, since rare-variant analyses are highly sensitive to genotype calling errors.

---

## 4. Export cleaned genotype calls from GenomeStudio

After review, export:

- final genotype report
- updated cluster positions / cluster file if modified
- SNP statistics
- sample statistics

It is strongly recommended to save a frozen copy of the final GenomeStudio project state so that downstream analyses can be traced back to the exact genotype-calling decisions used.

---

## 5. Convert the data into PLINK format

Once the cleaned genotype report has been exported, convert it into PLINK format for downstream QC and analysis.

The goal is to produce:

- `.ped/.map` or text input files initially, if needed
- then binary PLINK files:
  - `.bed`
  - `.bim`
  - `.fam`

Example:

```bash
plink --file exomechip_raw --make-bed --out exomechip_step1
```

---

## 6. Perform sample-level QC in PLINK

Sample-level quality control usually includes:

- **sample missingness / call rate**
- **sex concordance**
- **heterozygosity checks**
- **duplicate sample detection**
- **relatedness / kinship**
- **population structure / ancestry outlier detection**

Common filtering goals include removing samples with:

- excessive missing genotypes
- discordant reported vs genetic sex
- abnormal heterozygosity
- contamination signals
- unexpected duplicates or close relatives, depending on study design

Example missingness calculation:

```bash
plink --bfile exomechip_step1 --missing --out exomechip_missing
```

---

## 7. Perform variant-level QC in PLINK

Variant-level QC generally includes:

- SNP missingness
- allele frequency calculation
- Hardy-Weinberg equilibrium testing
- batch effect review
- removal of poorly performing markers

For rare-variant exome-chip studies, Hardy-Weinberg filtering should be interpreted carefully, since very rare variants may not behave like common GWAS SNPs in standard QC summaries.

Example commands:

```bash
plink --bfile exomechip_step1 --freq --out exomechip_freq
plink --bfile exomechip_step1 --hardy --out exomechip_hwe
```

---

## 8. Apply study-defined filters

Filtering thresholds should be tailored to the dataset, but common starting points include:

- sample missingness threshold: `--mind 0.02`
- SNP missingness threshold: `--geno 0.05`

Example:

```bash
plink --bfile exomechip_step1 \
      --mind 0.02 \
      --geno 0.05 \
      --make-bed \
      --out exomechip_step2
```

Additional filters may later include:

- ancestry-restricted analysis sets
- case/control-specific HWE filtering
- removal of monomorphic markers
- removal of technical duplicates
- plate or batch artifact filtering

---

## Example PLINK QC Workflow

```bash
# Convert to PLINK binary format
plink --file exomechip_raw --make-bed --out exomechip_step1

# Compute sample and SNP missingness
plink --bfile exomechip_step1 --missing --out exomechip_missing

# Calculate allele frequencies
plink --bfile exomechip_step1 --freq --out exomechip_freq

# Hardy-Weinberg equilibrium summary
plink --bfile exomechip_step1 --hardy --out exomechip_hwe

# Apply missingness filters
plink --bfile exomechip_step1 \
      --mind 0.02 \
      --geno 0.05 \
      --make-bed \
      --out exomechip_step2
```

---

## Suggested Directory Structure

```text
project/
├── data_raw/
│   ├── genomestudio_exports/
│   ├── manifests/
│   ├── cluster_files/
│   └── sample_sheets/
├── data_qc/
│   ├── plink_step1/
│   ├── plink_step2/
│   └── reports/
├── scripts/
│   ├── qc_plink.sh
│   ├── sample_qc.R
│   └── variant_qc.R
├── results/
│   ├── figures/
│   ├── tables/
│   └── cleaned_dataset/
└── README.md
```

---

## Common Pitfalls

### 1. Treating rare variants like common GWAS SNPs
Rare variants frequently require more manual inspection and should not be filtered or interpreted exactly like common markers.

### 2. Over-relying on automatic clustering
GenomeStudio’s default clustering may underperform on sparse genotype classes, especially when heterozygotes are extremely rare.

### 3. Using one-size-fits-all QC thresholds
Thresholds should be justified by study design, sample size, ancestry composition, and platform behavior.

### 4. Ignoring batch or plate effects
Exome-chip data can contain technical artifacts that mimic biological structure if plate, center, or processing batch is not tracked.

### 5. Failing to preserve reproducibility
Always retain:
- exported reports
- final cluster files
- PLINK logs
- filtering thresholds
- sample exclusion lists
- SNP exclusion lists

---

## Outputs

The expected outputs of preprocessing are:

- cleaned PLINK binary files:
  - `final.bed`
  - `final.bim`
  - `final.fam`
- QC summary files
- sample removal list
- SNP removal list
- logs of GenomeStudio cluster edits
- documentation of all filtering thresholds and decisions

---

## Notes

Because exome-chip studies focus on low-frequency and rare functional variants, preprocessing is not just a technical formality. The quality of rare-variant association testing depends heavily on accurate genotype calling, thoughtful cluster review, and carefully documented QC steps.

A strong preprocessing workflow should therefore prioritize:

- rare-variant cluster integrity
- reproducibility
- transparent filtering
- preservation of biological signal while removing technical noise

---

## Citation / Acknowledgment

If this workflow is used in a study or pipeline, please cite the relevant software and platform documentation for:

- Illumina GenomeStudio
- Illumina exome-chip platform
- PLINK

---

## Contact

For questions or extensions, adapt this workflow to your study design, phenotype structure, ancestry composition, and downstream rare-variant association framework.

