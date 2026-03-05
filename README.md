# Cytosine Content and Amino Acid Distribution in Prion-Forming Genes

Computational analysis comparing cytosine content, nucleotide/amino acid composition, and codon usage bias between prion-forming and non-prion-forming genes in the *Saccharomyces cerevisiae* genome. Includes a mutational scanning pipeline for generating mutant prion-domain libraries for downstream amyloid prediction (WALTZ).

## Overview

Prions are self-propagating protein conformations that can act as heritable elements of phenotypic change. In yeast, at least 18 proteins have been experimentally confirmed to form prions. This project investigates whether the DNA and protein sequence composition of prion-forming genes is systematically distinct from the rest of the yeast genome — with particular attention to **cytosine content** and its downstream consequences on amino acid usage and codon bias.

## Data

The analysis starts from the **S. cerevisiae ORF coding sequences** (FASTA format) obtained from the Saccharomyces Genome Database (SGD):

- **Source file:** `orf_coding_all_R64-4-1_20230830.fasta` (SGD R64-4-1 release)
- **Filtered to:** Verified ORFs only (5,254 genes)
- **Prion-forming genes (n=18):** SUP35, CYC8, RNQ1, URE2, NEW1, SWI1, MOD5, MOT3, GLN3, SFP1, SKY1, PUB1, PIN2, AZF1, VTS1, SNT1, RBS1, PSP1
- **Non-prion-forming genes (n=5,236):** All remaining verified ORFs

### Prion-Forming Domain (PrD) Boundaries

Each prion-forming gene has a known prion-forming domain. The boundaries used in this analysis are defined in the notebook:

| Gene  | PrD (amino acid positions) |
|-------|---------------------------|
| SUP35 | 1–135                     |
| CYC8  | 443–682                   |
| RNQ1  | 153–405                   |
| URE2  | 1–89                      |
| NEW1  | 1–153                     |
| SWI1  | 1–323                     |
| MOD5  | 194–215                   |
| MOT3  | 1–295                     |
| GLN3  | 166–242                   |
| SFP1  | 230–458                   |
| SKY1  | 353–491                   |
| PUB1  | 290–386                   |
| PIN2  | 221–282                   |
| AZF1  | 1–300                     |
| VTS1  | 1–420                     |
| SNT1  | 1–1226                    |
| RBS1  | 260–400                   |
| PSP1  | 50–400                    |

## Analysis Pipeline

The notebook (`prion_mutational_analysis.ipynb`) performs the following steps:

### 1. FASTA Parsing and Gene Annotation
- Parses the FASTA file and extracts gene ID, gene name, ORF classification, and coding sequence.
- Filters to verified ORFs and labels each gene as prion-forming (1) or non-prion-forming (0) based on curated gene lists.
- Validates all coding sequences (divisible by 3, starts with ATG, ends with a stop codon).

### 2. Nucleotide Composition
- Computes the percentage of each nucleotide (A, T, G, C) across the full coding sequence of every gene.

### 3. Translation
- Translates all coding DNA sequences to protein sequences using Biopython.

### 4. Amino Acid Composition
- Computes the percentage of each of the 20 standard amino acids in every protein.

### 5. Codon Usage Bias
- Counts every codon (as a percentage of total codons) for each gene, organized by the amino acid each codon encodes (e.g., `L_CTG`, `S_TCT`).

### 6. GC Content
- Computes overall GC content per gene.
- Computes the frequency of codons containing C or G at the 1st or 2nd codon position, as a proxy for how cytosine/guanine in template-strand positions influences amino acid identity.

### 7. Statistical Comparison (Resampling-Based Mann-Whitney U)
- Since the prion-forming group (n=18) is far smaller than the non-prion-forming group (n=5,236), a **resampling approach** is used:
  - For each of 10,000 iterations, 18 non-prion genes are randomly sampled.
  - A two-sided Mann-Whitney U test is performed for every feature (nucleotide %, amino acid %, codon %, GC content, etc.).
  - The resulting p-value distributions are plotted as histograms, color-coded by which group had the higher median.
- This approach avoids bias from unequal sample sizes and provides a distribution of significance rather than a single test.

### 8. Mutant Library Generation for WALTZ
- Generates systematic single-point mutant libraries across the prion-forming domains for downstream amyloid aggregation prediction using [WALTZ](http://waltz.switchlab.org/).
- **Amino acid mutations:** Substitutes a specified residue (e.g., every Q) with one or more target amino acids across the PrD, producing FASTA files per gene.
- **Nucleotide mutations:** Substitutes a specified nucleotide (e.g., every C→U) in the PrD coding sequence, translates the result, and records only non-synonymous changes.
- Outputs per-gene FASTA files with mutation labels (e.g., `SUP35;Q15L`) ready for batch WALTZ submission.

## Requirements

All code was written for **Google Colab** in a Python 3 environment.

### Dependencies

| Package     | Purpose                            |
|-------------|------------------------------------|
| pandas      | Data manipulation                  |
| numpy       | Numerical operations               |
| matplotlib  | Plotting                           |
| seaborn     | Statistical visualization          |
| biopython   | FASTA parsing, sequence translation|
| scipy       | Statistical tests                  |

Install Biopython in Colab with:
```bash
pip install biopython
```

## Usage

1. Open `prion_mutational_analysis.ipynb` in [Google Colab](https://colab.research.google.com/).
2. Mount your Google Drive when prompted.
3. Place the FASTA file (`orf_coding_all_R64-4-1_20230830.fasta`) in your Google Drive at the `fasta_path` variable accordingly.
4. Run all cells sequentially. Statistical comparison figures and mutant FASTA files will be saved to the specified output directories on Google Drive.

## Repository Structure

```
Cytosine_Prion/
├── README.md
└── prion_mutational_analysis.ipynb   # Main analysis notebook
```

## License

This project is associated with the [Chen Lab at USC](https://github.com/ChenLabUSC).
