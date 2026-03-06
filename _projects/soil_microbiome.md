---
layout: project
title: Functional Redundancy in Soil Microbiomes under Cover Crop Agroecosystems
description: Genome-resolved metagenomics and metatranscriptomics reveal microbial functional redundancy across agricultural soils under different cover crop systems.
img: assets/img/Plot_Layout .png
importance: 6
category: work
related_publications: true
---

#### Functional Redundancy in Soil Microbiomes under Cover Crop Agroecosystems

**🧠 Background & Motivation**

Agricultural soils host diverse microbial communities that regulate **nutrient cycling, plant productivity, and ecosystem stability**. Cover crops are widely used to improve soil health and sustainability, yet the mechanisms by which microbial communities maintain ecosystem functions under different plant systems remain poorly understood.

One key concept is **functional redundancy**, where multiple microbial taxa share similar metabolic capabilities, enabling ecosystems to maintain function even when community composition changes.

This project investigates **microbial community structure and functional redundancy in soils under different cover crop species** using genome-resolved metagenomics and metatranscriptomics.

---

#### 🎯 Research Questions & Objectives\*\*

\*_📌 Study 1: Microbial Community Structure in Agricultural Soils_

- What microbial lineages dominate soils under different cover crop systems?
- How does microbial diversity vary across agricultural treatments?

\*_📌 Study 2: Genome-Resolved Soil Microbiome Reconstruction_

- Which microbial genomes can be reconstructed from soil metagenomes?
- What metabolic traits are encoded in dominant soil microbial taxa?

\*_📌 Study 3: Functional Redundancy in Soil Ecosystems_

- To what extent do multiple microbial taxa share metabolic functions?
- How does redundancy contribute to ecosystem stability in agroecosystems?

---

#### 📊 Dataset

This study generated a comprehensive soil microbiome dataset including:

- **30 soil metagenomes**
- **21 metatranscriptomes**
- **355 metagenome-assembled genomes (MAGs)**

These data provide a resource for studying **soil microbial ecology, functional redundancy, and microbial interactions in agricultural systems**.

---

#### 🧬 Microbial Diversity

Taxonomic classification using **GTDB-Tk (v2.6.1)** identified:

- **345 bacterial MAGs**
- **10 archaeal MAGs**

Dominant bacterial lineages included:

- Thermoleophilia (57 MAGs)
- Alphaproteobacteria (55 MAGs)
- Terriglobia (44 MAGs)
- Gemmatimonadetes (31 MAGs)
- Vicinamibacteria (28 MAGs)
- Gammaproteobacteria (23 MAGs)
- Actinomycetes (20 MAGs)

Additional taxa included members of:

- Acidimicrobiia
- Nitrospiria
- Verrucomicrobiae
- Chloroflexia
- Ktedonobacteria
- Bacteroidia

Archaeal MAGs were primarily affiliated with **Nitrososphaeria**, a key lineage involved in ammonia oxidation and nitrogen cycling in soils.

---

#### 👨‍🔬 My Role

My primary contribution focused on **genome-resolved metagenomics, computational workflow development, and functional redundancy analysis**, including:

- Co-assembly of soil metagenomes using **MEGAHIT**
- **Supervised genome binning using SemiBin**
- Genome quality assessment and taxonomic classification
- Development of a **custom workflow to quantify functional redundancy from metagenomic datasets**

---

#### ⚙️ Functional Redundancy Workflow Development

To quantify functional redundancy across soil microbial communities, I developed a **custom computational wrapper** that integrates metagenomic and metatranscriptomic datasets.

The workflow includes:

1. **Custom database construction**
   - Genome database built using **Struo2**
   - Reference taxonomy derived from **GTDB-Tk**

2. **Functional trait profiling**
   - Metabolic trait tables generated using **HUMAnN3**

3. **Genome selection**
   - Identification of ~**2,000 dominant genomes** from the trait table
   - Custom **Python and Perl scripts** used to extract representative genomes

4. **Abundance estimation**
   - Metagenomic and metatranscriptomic reads mapped against genome database
   - Species abundance tables generated from mapping outputs

5. **Functional redundancy quantification**
   - Species abundance integrated with trait tables
   - Redundancy metrics calculated across microbial taxa and metabolic traits

The workflow has been **successfully tested using pilot datasets** and provides a scalable framework for linking microbial community composition to ecosystem functions.

---

**🛠 Methods & Tools**

\*_Metagenomics_

- MEGAHIT (co-assembly)
- SemiBin (supervised genome binning)
- CheckM
- GTDB-Tk

\*_Functional Profiling_

- HUMAnN3
- Custom Python and Perl scripts

\*_Database Construction_

- Struo2 genome database generation

\*_Analysis_

- R (phyloseq, vegan)
- HPC-based reproducible pipelines

---

#### 🌱 Perspective

This project advances our understanding of **how microbial communities maintain functional stability in agricultural soils**. By integrating **genome-resolved metagenomics with functional redundancy analysis**, the study provides new insights into microbial contributions to **soil health, nutrient cycling, and agroecosystem sustainability**.

The computational workflow developed in this work offers a **scalable framework for linking microbial genomes, metabolic traits, and ecosystem functions**, enabling future studies on
microbiome-driven agricultural resilience.

#### Output

Giani, N., John, J., & Campbell, B. (2026). Metagenomes, metatranscriptomes, and metagenome- assembled genomes (MAGs) collected from soils under different cover crop species. Submitting
Microbiology Resource Announcements. Metagenomes (n = 21), metatranscriptomes (n = 21), and 355 metagenome-assembled genomes (MAGs) available on NCBI.

#### Refrence

**Dr. Barbara J. Campbell**
Dean’s Distinguished Professor
Department of Biological Sciences, Clemson University
Email: bcampb7@clemson.edu

**Nichole Giani**
PhD Candidate | Campbell Lab
Microbiology
Clemson University
Email:ngiani@g.clemson.edu
