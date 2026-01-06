---
layout: project
title: "Bacteriophages and Auxiliary Metabolic Genes of the Chesapeake and Delaware Bays"
description: "Integrative viromics using metagenomes, metatranscriptomes, and viral fractions to characterize viral taxonomy, auxiliary metabolic genes, host interactions, and ecosystem function in estuarine systems."
img: assets/img/vir_all.png
importance: 2
category: work
related_publications: true
---

## 🧠 Background & Motivation

Viruses are key regulators of microbial community structure and ecosystem function in marine environments. Beyond their role in host mortality, many bacteriophages encode **auxiliary metabolic genes (AMGs)**—genes that overlap with host metabolic pathways and can reprogram host metabolism to enhance viral replication. Through this process, viruses influence **carbon cycling, nutrient turnover, and energy flow**, ultimately contributing to ecosystem homeostasis.

In estuarine systems such as the **Chesapeake and Delaware Bays**, microbial communities experience strong seasonal and anthropogenic variability. The acquisition and expression of AMGs may provide a mechanism by which viral populations support microbial adaptation to **environmental fluctuations and climate-driven stressors**.

In this project, I integrated **metagenomic, metatranscriptomic, and viral fraction datasets** from seasonal sampling campaigns to characterize **viral diversity, activity, and metabolic potential**. In addition to assembled viruses from whole-community datasets, this work leverages an **independent, published viral fraction dataset from the Delaware Bay**, enabling direct comparison between **cell-associated and free viral communities** from the same ecosystem.

---

## 🎯 Research Questions & Objectives

- What is the **taxonomic composition** of DNA and RNA viruses in the Chesapeake and Delaware Bays?
- How does viral diversity inferred from **metagenomes and metatranscriptomes** compare to that from **viral fraction datasets**?
- Which **auxiliary metabolic genes (AMGs)** are encoded by estuarine viruses, and which metabolic pathways do they target?
- Are AMGs **actively expressed**, and do expression patterns vary seasonally or between bays?
- Can viral **host associations** be inferred using genome-based and gene-sharing approaches?
- How do estuarine viral communities compare with **existing coastal and marine virome datasets**?

---

## 👨‍🔬 My Role

- Designed the **comparative viromics framework** integrating MG, MT, and viral fraction data
- Identified and curated viral contigs from metagenomes and metatranscriptomes
- Performed viral **taxonomy assignment and vOTU clustering**
- Annotated and classified **auxiliary metabolic genes (AMGs)**
- Quantified viral and AMG **abundance and transcriptional activity**
- Conducted **host prediction and gene-sharing network analyses**
- Led comparative analyses with previously published estuarine virome datasets

---

## 🧩 Challenges & Solutions

**Challenge 1:** Distinguishing viral contigs from cellular sequences in complex metagenomic assemblies  
**Solution:** Applied multiple viral detection tools and quality filters, followed by manual curation and validation against viral reference databases.

---

**Challenge 2:** Identifying true AMGs while avoiding false positives from host contamination  
**Solution:** Used **AMG-specific annotation frameworks** with metabolic context checks, flanking gene inspection, and pathway-level validation.

---

**Challenge 3:** Integrating whole-community metagenomes with independently generated viral fraction datasets  
**Solution:** Standardized vOTU clustering, taxonomy, and functional annotation pipelines across datasets to enable **direct cross-study comparisons**.

---

**Challenge 4:** Linking viruses to potential microbial hosts in the absence of cultured references  
**Solution:** Combined **gene-sharing networks, sequence similarity, and taxonomy-informed inference** to predict virus–host associations.

---

## 🛠 Methods & Tools

### Data & Sequencing

- Shotgun metagenomics
- Metatranscriptomics
- Viral fraction metagenomes (Delaware Bay, published dataset)
- Read mapping for viral abundance and expression
- Comparative viromics across datasets

---

### Virome Analysis & Annotation

**Viral Identification & Quality Control**

- VirSorter2
- VITAP
- CheckV

**Taxonomy & Classification**

- geNomad
- vConTACT2

**AMG Detection & Functional Annotation**

- DRAM-v
- VIBRANT
- eggNOG-mapper

**Host Prediction & Networks**

- Gene-sharing networks
- Similarity-based host inference

**Mapping & Quantification**

- Bowtie2
- SAMtools

---

### Languages & Workflow

- Python
- Bash
- R
- Snakemake
- SLURM (HPC scheduling)

---

## 📄 Publications

- \*_Manuscript in preparation_

---

## 🎤 Conferences & Abstracts

- **Bacteriophages and Auxiliary Metabolic Gene interactions in microbial adaptation in the Chesapeake and Delaware Bays**  
  Barbara J. Campbell, Kasey Kiser, Sam Stuckert, **Jojy John**  
  ASM Microbe 2025, Los Angeles, USA

---

## 🧑‍🔬 Collaborators / References

**Dr. Barbara J. Campbell**  
Dean’s Distinguished Professor  
Department of Biological Sciences, Clemson University  
Email: bcampb7@clemson.edu

---

## 🖼 Image Gallery

<div class="row">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/virome_pipeline.png" title="Integrated virome analysis pipeline from metagenomes, metatranscriptomes, and viral fractions" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/amg_categories.png" title="Functional categories of auxiliary metabolic genes identified in estuarine viruses" class="img-fluid rounded z-depth-1" %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid loading="eager" path="assets/img/viral_network.png" title="Viral gene-sharing network and host association framework" class="img-fluid rounded z-depth-1" %}
  </div>
</div>
