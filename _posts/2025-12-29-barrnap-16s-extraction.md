---
layout: post
title: "Extracting Full-Length 16S rRNA from Bacterial Genomes Using Barrnap"
date: 2025-12-29
description: "A quick and reproducible workflow to extract 16S rRNA genes from assembled genomes and validate taxonomy using NCBI BLAST."
comments: true
giscus_comments: true
featured: false
permalink: /blog/barrnap-16s-extraction/
---

#### 🧬 Extracting Full-Length 16S rRNA from Bacterial Genomes Using Barrnap

Whole-genome sequencing has transformed microbial taxonomy, with genome-wide metrics such as ANI, AAI, and dDDH now serving as the gold standard for species delineation.
However, the 16S rRNA gene remains an essential taxonomic anchor, especially for linking isolate genomes to classical taxonomy and amplicon-based studies.

In this short post, I walk through a simple and reproducible workflow to:

1. Extract full-length 16S rRNA genes from an assembled bacterial genome

2. Validate taxonomic placement using NCBI BLAST

**🔧 Tool Overview: Barrnap**

Barrnap (BAsic Rapid Ribosomal RNA Predictor) is a fast and lightweight tool that identifies ribosomal RNA genes (5S, 16S, and 23S) in bacterial and archaeal genomes using HMM-based models.

Why Barrnap?

● Extremely fast (seconds per genome)

● Accurate for bacterial and archaeal rRNAs

● Ideal for complete genomes and high-quality drafts

● Outputs standard GFF annotations

**🔧 Installation (Linux / Conda)**

```bash
conda install -c bioconda barrnap
```

▶️ You’ll also need BEDTools for sequence extraction:

```bash
conda install -c bioconda bedtools
```

#### ▶️ Step 1: Predict rRNA genes with Barrnap

```bash
barrnap --kingdom bac genome.fasta > genome_rrna.gff
```

\*\*Replace genome.fasta with your genome file

**What this step does**

● Scans the genome for rRNA genes using bacterial HMM profiles

● Identifies coordinates for 5S, 16S, and 23S rRNA genes

● Outputs results in GFF format with genomic locations

#### 🧬 Step 2: Extract the 16S rRNA sequence

Once rRNA coordinates are available, extract the sequence directly from the genome using bedtools:

```bash
bedtools getfasta \
  -fi genome.fasta \
  -bed genome_rrna.gff \
  -fo genome_rrna.fasta
```

The output FASTA typically contains:

● One or more rRNA sequences

● Full-length or near full-length 16S rRNA genes (≈1,500 bp)

If multiple rRNA operons are present, all will be extracted.
hint : I always select the longest one to proceed

#### 🔍 Step 3: Identify the closest relatives using NCBI BLAST

The extracted 16S rRNA sequence can now be compared against public databases using NCBI BLAST.

**Option A: Web-based BLAST (quickest)**

1. Go to: https://blast.ncbi.nlm.nih.gov

2. Select BLASTn

3. Paste the 16S sequence from PC_D3_3_rrna.fasta

4. Database: nt

5. Program selection: Highly similar sequences (megablast)

**Option B: Command-line BLAST (reproducible)**

```bash
blastn \
  -query genome_rrna.fasta \
  -db nt \
  -out genome_16S_blast.tsv \
  -outfmt 6 \
  -max_target_seqs 10 \
  -evalue 1e-20
```

\*\*To run BLAST from the command line, you need the NCBI BLAST+ toolkit installed and access to a nucleotide database (e.g., nt). BLAST+ can be installed via Conda, and the nt database can either be downloaded locally or accessed through a preconfigured BLAST database path on an HPC system.

#### Step 4 📊 How to interpret BLAST results

Key BLAST columns to focus on:

| Metric              | What it means                    |
| ------------------- | -------------------------------- |
| % identity          | Sequence similarity to reference |
| Alignment length    | Length of the matched region     |
| Query coverage      | Fraction of 16S gene aligned     |
| E-value             | Statistical significance         |
| Subject description | Closest named organism           |

**Typical interpretation guidelines**

● ≥99% identity → likely same species

● 98.7–99% identity → borderline species-level similarity

● <98.7% identity → distinct species

● <95% identity → possibly different genus

\*_It is important to note that 16S alone is not sufficient for species designation, but it provides strong supporting evidence when combined with genome-wide metrics._

#### 🧠 Why include 16S in a genome-based study?

Even in the genomics era, 16S rRNA sequences are still valuable for:

● Connecting isolate genomes to historical literature

● Comparing isolates to amplicon-based surveys

● Supporting taxonomic claims in Genome Resource Announcements

● Providing a familiar reference for non-genomics audiences

**In practice, 16S supports taxonomy, while ANI, AAI, and dDDH define it.**

#### ✅ Take-home messages

● Barrnap provides a fast and reliable way to extract rRNA genes from genomes

● Full-length 16S rRNA sequences remain an important taxonomic reference

● BLAST results should be interpreted alongside genome-wide comparisons

● This workflow takes minutes and fits naturally into any isolate genome analysis

![16s-from-genome](/assets/img/br.png)
