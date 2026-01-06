---
layout: project
title: "Functional Redundancy and Metabolic Flexibility of Microbial Communities in Two Mid-Atlantic Bays"
description: "Genome-resolved metagenomic and metatranscriptomic analysis of functional redundancy and metabolic flexibility in estuarine microbial communities from the Chesapeake and Delaware Bays."
img: assets/img/project10.jpg
importance: 1
category: work
related_publications: true
---

## 🧠 Background & Motivation

Species and communities respond differently to environmental stress depending on the **extent of functional redundancy** within the system. In microbial ecosystems, functional redundancy (FRed) can buffer ecosystem processes against disturbance by allowing multiple taxa to perform similar functions. However, the **relationship between functional redundancy, metabolic flexibility, and ecosystem resilience remains poorly understood**, particularly in dynamic coastal systems.

Estuaries are ideal natural laboratories for studying these relationships. They experience rapid environmental fluctuations driven by **seasonality, nutrient inputs, salinity gradients, hypoxia, and anthropogenic stress**, often leading to long-term ecosystem degradation. Comparing geographically proximate yet biologically distinct estuaries allows disentangling how community composition and function respond to similar stressors.

In this project, I integrated **metagenomics, metatranscriptomics, and genome-resolved analyses** across seasonal samples to quantify microbial composition, metabolic potential, activity, and **functional redundancy** in the **Chesapeake and Delaware Bays (USA)**. This work combines ecological theory with large-scale multi-omics data and statistical modeling to explore how microbial communities maintain ecosystem function under disturbance.

---

## 🎯 Research Questions & Objectives

- How does **functional redundancy vary across seasons and between estuarine systems**?
- Do microbial communities rely on **metabolic flexibility** (e.g., photoheterotrophy, lithoheterotrophy) to maintain ecosystem processes under stress?
- Are specific metabolic pathways (e.g., energy acquisition) more **functionally redundant** than others?
- How do **DNA-based metabolic potential** and **RNA-based activity** differ in shaping functional redundancy?
- Are communities with high functional diversity but low redundancy more **vulnerable to disturbance**?
- Does higher functional redundancy necessarily translate to **greater ecosystem resilience**?

---

## 👨‍🔬 My Role

This project represents my **primary postdoctoral research**.

- Designed the **conceptual framework** linking functional redundancy, metabolic flexibility, and ecosystem resilience
- Performed **metagenomic and metatranscriptomic analyses** across seasonal datasets
- Quantified **functional redundancy metrics** at gene, pathway, and metabolic-module levels
- Conducted **genome-resolved metagenomics**, including MAG reconstruction, annotation, and taxonomy
- Led **statistical modeling**, regression analyses, and multivariate ecological analyses
- Interpreted results in the context of ecological theory and estuarine biogeochemistry
- Primary author of the manuscript and conference presentations

---

## 🧩 Challenges & Solutions

**Challenge 1:** Transitioning to fully independent, large-scale bioinformatics without senior technical support  
**Solution:** Systematically evaluated and benchmarked multiple pipelines, optimizing workflows using a subset of samples before scaling analyses to full datasets.

---

**Challenge 2:** Lack of established, standardized methods for quantifying microbial functional redundancy  
**Solution:** Developed a customized analytical framework through extensive literature review, interdisciplinary collaboration, and iterative discussions with ecologists and statisticians.

---

**Challenge 3:** Interpreting functional redundancy in the absence of directly comparable prior studies  
**Solution:** Integrated **environmental metadata**, diversity metrics, and metabolic pathway analyses to contextualize FRed patterns within known estuarine gradients and ecological theory.

---

**Challenge 4:** Integrating DNA-based potential with RNA-based activity across uneven sequencing depths  
**Solution:** Applied normalization strategies, pathway-level aggregation, and comparative DNA:RNA analyses to robustly link metabolic capacity and expression.

---

## 🛠 Methods & Tools

### Data & Sequencing

- Shotgun metagenomics
- Metatranscriptomics
- Metagenome-assembled genomes (MAGs)
- Read mapping and expression quantification
- Functional redundancy modeling

### Bioinformatics & Statistics

- Genome annotation and pathway reconstruction
- Taxonomic classification and MAG quality assessment
- Multivariate statistics and regression modeling
- Diversity metrics (alpha, beta, functional diversity)

### Tools & Software

- MetaWRAP
- GTDB-Tk
- DRAM / DRAM-v
- CoverM
- Bowtie2
- SAMtools
- R (tidyverse, vegan, ggplot2, lme4)

### Languages & Workflow

- Python
- Bash
- R
- Snakemake
- SLURM (HPC scheduling)

---

## 📄 Publications

- **Functional redundancy and metabolic flexibility of microbial communities in two Mid-Atlantic bays**  
  _ISME Communications_ — **under revision (Round 2)**  
  Preprint available upon request.

---

## 🎤 Conferences & Talks

- **Functional redundancy and metabolic flexibility of microbial communities in two Mid-Atlantic bays**  
  Jojy John, Maximiliano Ortiz, Barbara J. Campbell  
  ASLO Aquatic Sciences Meeting, Charlotte, USA, 2025

- **Microbial functional redundancy in response to substrate and energy utilization in estuarine ecosystems**  
  Jojy John, Maximiliano Ortiz, Pierre Ramond, Barbara J. Campbell  
  ISME19, Cape Town, South Africa, 2024

- **Does the microbiome insure ecosystem function?**  
  Jojy John, Maximiliano Ortiz, Barbara J. Campbell  
  Clemson University 3rd Postdoctoral Symposium, South Carolina, USA, 2024

---

## 🧑‍🔬 Collaborators / References

**Dr. Barbara J. Campbell**  
Dean’s Distinguished Professor  
Department of Biological Sciences, Clemson University  
Email: bcampb7@clemson.edu

**Dr. Pierre Ramond**  
Postdoctoral Scientist  
Institut de Ciències del Mar (ICM-CSIC)  
Email: pierre@icm.csic.es

---

## 🖼 Image Gallery

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/fred_framework.jpg" title="Conceptual framework of functional redundancy and metabolic flexibility" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/fred_heatmap.jpg" title="Functional redundancy patterns across seasons and bays" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/fred_dnarna.jpg" title="Comparison of genomic potential and transcriptional activity" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
