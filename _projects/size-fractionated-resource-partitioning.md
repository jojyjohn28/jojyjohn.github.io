---
layout: project
title: "Resource Partitioning and Co-occurrence of Size-Fractionated Microbiomes in the Delaware and Chesapeake Bays"
description: "Integrative metagenomic and metatranscriptomic analysis of free-living and particle-associated microbial communities to understand resource partitioning, co-occurrence patterns, and functional redundancy in estuarine ecosystems."
img: assets/img/flvsPA.png
importance: 4
category: work
related_publications: false
---

## 🧠 Background & Motivation

Microbial communities in aquatic ecosystems are structured into **free-living (FL)** and **particle-attached (PA)** fractions, representing distinct ecological niches with different access to substrates, nutrients, and microenvironments. These size-fractionated communities play complementary roles in **carbon cycling, nutrient transformation, and ecosystem functioning**, yet their **community assembly, metabolic strategies, and interaction networks remain poorly resolved**, particularly under dynamic environmental conditions.

The **Chesapeake and Delaware Bays**, two geographically proximate but environmentally distinct estuaries along the U.S. Atlantic coast, differ in **salinity gradients, particle load, light availability, and anthropogenic influence**. These contrasts provide a natural framework to investigate how **resource partitioning, substrate uptake, and microbial interactions** shape FL and PA communities across seasons.

In this project, I applied **read-based metagenomic and metatranscriptomic analyses** across spring, summer, and fall to quantify **taxonomic composition, metabolic potential, activity, co-occurrence patterns, and functional redundancy** in FL and PA microbial communities.

---

## 🎯 Research Questions & Objectives

- How do **free-living and particle-attached microbiomes differ** in taxonomic and functional diversity?
- Do FL and PA communities exhibit **distinct co-occurrence and interaction networks**?
- Does **substrate uptake and resource partitioning** drive niche differentiation between size fractions?
- How does **functional redundancy** vary between FL and PA communities and across seasons?
- Are active (RNA-based) interaction networks different from total (DNA-based) networks?

---

## 👨‍🔬 My Role

This project is part of my **primary postdoctoral research program**.

- Designed the **conceptual framework** linking size fractionation, resource partitioning, co-occurrence, and functional redundancy
- Performed **read-based metagenomic and metatranscriptomic analyses** across seasonal datasets
- Quantified **substrate uptake pathways** and metabolic specialization
- Constructed **co-occurrence networks** for FL and PA communities
- Developed a **read-based functional redundancy matrix** adapted from MAG-based frameworks
- Integrated **total (DNA) and active (RNA)** community analyses
- Conducted **differential expression analysis** and **MTX modeling**
- Led data interpretation and visualization

---

## 🧩 Challenges & Solutions

**Challenge 1:** Distinguishing true differential expression from differences driven by **gene copy number variation**  
**Solution:** Implemented **MTX modeling** and DNA:RNA normalization approaches to decouple transcriptional activity from genomic abundance.

---

**Challenge 2:** Lack of an established **functional redundancy framework for read-based datasets**  
**Solution:** Transformed and adapted **MAG-based functional redundancy matrices** for read-level data, enabling pathway-level FRed quantification without relying solely on genome bins.

---

**Challenge 3:** Inconsistent taxonomic lineage and trait assignments across profiling tools  
**Solution:** Built a **GTDB-Tk–based reference framework** to harmonize outputs from HUMAnN3, Kaiju, mOTUs, and MetaPhlAn, integrating **Pfam- and KO-based functional annotations**.

---

**Challenge 4:** Inferring robust microbial interaction networks from compositional data  
**Solution:** Applied **composition-aware network methods** and cross-validated co-occurrence patterns across seasons, size fractions, and DNA/RNA datasets.

---

## 🛠 Methods & Tools

### Data & Sequencing

- Shotgun metagenomics
- Metatranscriptomics
- Read mapping and expression quantification
- Functional redundancy (FRed) modeling
- MTX modeling
- Co-occurrence and network analysis

---

### Bioinformatics & Network Analysis

**Taxonomy & Function**

- Kaiju
- mOTUs
- HUMAnN3
- eggNOG-mapper
- dbCAN
- DIAMOND

**Mapping & Quantification**

- Bowtie2
- SAMtools

**Network Analysis**

- SparCC
- SPIEC-EASI
- igraph (R)

---

### Languages & Workflow

- Python
- Bash
- R
- Snakemake
- SLURM (HPC scheduling)

---

## 📄 Publications

- **Resource partitioning and co-occurrence shape functional redundancy of size-fractionated microbiomes in the Delaware and Chesapeake Bays**  
  _Manuscript in preparation._

---

## 🎤 Conferences & Talks

- **Metabolic flexibility, secondary metabolism, and seasonal dynamics in particle-associated and free-living microorganisms in the Chesapeake and Delaware Bays**  
  Jojy John, Mir A. Ahmed, Barbara J. Campbell  
  ASM Microbe 2025, Los Angeles, USA

---

## 🧑‍🔬 Collaborator / Reference

**Dr. Barbara J. Campbell**  
Dean’s Distinguished Professor  
Department of Biological Sciences, Clemson University  
Email: bcampb7@clemson.edu

---

## 🖼 Image Gallery

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/fl_pa_concept.png" title="Conceptual framework of free-living and particle-associated microbial niches" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/fl_pa_network.png" title="Co-occurrence networks of size-fractionated microbiomes" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/fl_pa_fred.png" title="Functional redundancy patterns across size fractions and seasons" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
