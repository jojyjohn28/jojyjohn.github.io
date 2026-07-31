---
layout: project
title: Co-Metabolism and Co-Expression Networks of Hydrocarbon-Degrading Estuarine Microbiomes
description: Genome-resolved analysis of hydrocarbon-degrading MAGs across two Mid-Atlantic estuaries, linking metabolic potential, co-energy pathways, and co-metabolism networks using metagenomic and metatranscriptomic data.
img: assets/img/co_meta.jpeg
importance: 7
category: work
related_publications: true
---

#### Co-Metabolism and Co-Expression Networks of Hydrocarbon-Degrading Estuarine Microbiomes

**🧠 Background & Motivation**

Hydrocarbon degradation in natural ecosystems is rarely an isolated function. In estuarine microbiomes, it is embedded within broader metabolic frameworks that support **energy conservation, redox balance, and carbon flow**. Understanding these linked processes is essential for explaining how microbial communities respond to environmental gradients and sustain ecosystem function in dynamic coastal systems.

This project investigates **hydrocarbon-degrading microbial populations in two Mid-Atlantic estuaries** using a **genome-resolved, multi-omics framework**. The goal is to move beyond simply identifying hydrocarbon degradation genes and instead ask how these organisms are metabolically structured, what energy systems co-occur with degradation, and which pathways are actively expressed together under natural environmental conditions.

This work is part of a broader collaborative project on estuarine hydrocarbon-degrading microbiomes, and I am actively **mentoring Dinuka** in the main project while contributing to the genome-resolved metabolic and co-expression analyses.

---

#### 🎯 Research Questions & Objectives

\*_📌 Study 1: Metabolic Potential of Hydrocarbon-Degrading MAGs_

- Which additional metabolic functions are encoded in MAGs that contain hydrocarbon degradation genes?
- Do hydrocarbon degraders share a common metabolic blueprint, or do they differ across taxa?

\*_📌 Study 2: Co-Energy Metabolism_

- Which energy generation and conservation pathways co-occur with hydrocarbon degradation in the same genome?
- How do these pathways vary across redox niches in estuarine environments?

\*_📌 Study 3: Co-Metabolism and Co-Expression_

- Which metabolic pathways are actively expressed together with hydrocarbon degradation genes?
- Can co-expression patterns reveal distinct ecological strategies among hydrocarbon-degrading taxa?

---

#### 👨‍🔬 My Role

My contribution focuses on **genome-resolved metabolism, computational workflow development, and mentoring**, including:

- Mentoring **Dinuka** as part of the broader hydrocarbon degrader project
- Identifying hydrocarbon-degrading MAGs based on key marker genes
- Building workflows to assess **metabolic potential, co-energy metabolism, and co-metabolism**
- Integrating **metagenomic and metatranscriptomic data** at the MAG level
- Developing **R-based visualization pipelines** for heatmaps and bipartite networks
- Interpreting co-expression and metabolic integration patterns across taxa and samples

---

#### 🧬 Analytical Framework

This project addresses three related but distinct questions:

1. **Metabolic Potential**  
   What additional metabolic capabilities are encoded in hydrocarbon-degrading MAGs?  
   → Based on **gene presence/absence**

2. **Co-Energy Metabolism**  
   Which energy pathways co-occur with hydrocarbon degradation?  
   → Based on **energy-related marker genes**

3. **Co-Metabolism**  
   Which metabolic pathways are actively expressed together in the same genome?  
   → Based on **gene expression patterns from metatranscriptomes**

Together, these analyses connect **what is possible** to **what is actively used**.

---

#### 🛠 Methods & Tools

\*_Genome-Resolved Analysis_

- MAG screening for hydrocarbon degradation markers (e.g., **alkB, bssA, assA**)
- Protein prediction using **Prodigal**
- Marker gene annotation using **DIAMOND**
- Genome-level metabolic potential summarization

\*_Metabolic Marker Framework_

Curated marker proteins were used to detect pathways related to:

- Aerobic and anaerobic respiration
- Sulfur cycling
- Nitrogen cycling
- Carbon fixation
- Phototrophy
- Fermentation
- Central carbon metabolism

\*_Expression Analysis_

- MAG-level gene expression profiling from metatranscriptomic datasets
- Category-level organization of genes into:
  - Hydrocarbon degradation
  - C1 metabolism
  - Complex carbon degradation
  - Fermentation
  - Central carbon metabolism

\*_Visualization & Statistics_

- **R**
- **pheatmap**
- **igraph**
- **ggraph**
- Custom scripts for category standardization, matrix generation, and network construction

---

#### 🧩 Challenges & Solutions\*\*

**Challenge 1:** Distinguishing metabolic potential from active co-metabolism  
**Solution:** Separated genomic presence/absence analyses from metatranscriptomic expression-based analyses, allowing clearer interpretation of encoded versus expressed functions.

**Challenge 2:** Inconsistent category naming and complex expression matrices  
**Solution:** Developed standardized data-cleaning and recoding steps to harmonize pathway categories before heatmap and network construction.

**Challenge 3:** Hidden functional differences among hydrocarbon degraders  
**Solution:** Used **MAG-resolved analyses** rather than bulk community summaries, revealing taxon-specific metabolic integration and ecological strategies.

---

#### 🌊 Key Insights

- Hydrocarbon degradation is not a standalone process but is embedded within **broader metabolic and energetic frameworks**
- Hydrocarbon-degrading MAGs differ substantially in their associated **respiration, sulfur, nitrogen, and carbon pathways**
- Co-metabolism patterns suggest **distinct ecological strategies** among taxa, including flexible versus highly integrated metabolic lifestyles
- Genome-resolved approaches reveal structure and function that would be masked in bulk metagenomic or metatranscriptomic analyses

---

#### 🌍 Perspective

This project advances our understanding of how hydrocarbon-degrading microorganisms function in **estuarine ecosystems shaped by environmental gradients and anthropogenic inputs**. By linking **metabolic potential, co-energy pathways, and active co-metabolism**, the work provides a more complete view of the ecological roles of hydrocarbon degraders in coastal biogeochemistry.

More broadly, this framework demonstrates how **genome-resolved metagenomics and metatranscriptomics** can be used to uncover hidden metabolic structure in environmental microbiomes, with relevance to **functional ecology, pollution response, and ecosystem resilience**.

#### Outcome

Patabandige, D. L. J., John, J., Ortiz, M., & Campbell, B. J. (2026). Environmental gradients shape the
hydrocarbon-degrading microbiome in two Mid-Atlantic bays. Submitted (ISME Communications).
Preprint available upon request

#### Reference

**Dr. Barbara J. Campbell**, 📧 bcampb7@clemson.edu

**Dinuka Lakmali Jayasuriya Patabandige**
PhD Student, Clemson University
📧 djayasu@g.clemson.edu

**Read More at https://jojyjohn28.github.io/blog/genome-resolved-metabolism-co-metabolism/**
