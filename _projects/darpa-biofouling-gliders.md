---
layout: project
title: "Microbial Biofouling Interactions on Autonomous Navy Gliders"
description: "Genome-resolved and metagenomic analysis of marine biofilm communities on autonomous underwater vehicles to design a stable, invasion-resistant synthetic biofilm."
img: assets/img/project11.jpg
importance: 3
category: work
related_publications: true
---

# Microbial Biofouling Interactions on Autonomous Navy Gliders

** 🧠 Background & Motivation**

Biofouling is the process by which microorganisms and multicellular organisms colonize submerged surfaces, negatively impacting marine vessels by increasing drag and reducing operational efficiency. This challenge is particularly critical for **battery-powered Unmanned Underwater Vehicles (UUVs)**, including **Autonomous Navy Gliders**, where increased drag leads to significant energy loss and reduced mission duration.

Rather than relying on toxic antifouling coatings, this project explores an **eco-friendly alternative**: engineering a **smooth, invasion-resistant, and stable synthetic microbial community (Eco-Coating)** composed of naturally occurring marine bacteria. By understanding the **genomic, metabolic, and biofilm-forming traits** of native biofouling communities, this work aims to rationally design microbial consortia that outcompete harmful fouling organisms while maintaining surface stability.

---

\**🎯 Research Questions & Objectives*8

- What microbial taxa colonize the **body and tail sections** of autonomous underwater vehicles?
- Which **genes and pathways** are associated with biofilm formation, EPS production, and antagonistic activity?
- How do isolate genomes and metagenome-assembled genomes (MAGs) inform the design of a **synthetic biofilm community**?
- Can genome-resolved traits predict **community stability and antifouling potential**?

---

**👨‍🔬 My Role**

- Provided **full bioinformatics support** for the project
- Performed **metagenomic and genome-resolved analyses** of UUV-associated biofilms
- Conducted **whole-genome assembly, hybrid assembly, and long-read assembly** of >100 bacterial isolates
- Genome annotation, functional screening, and comparative genomics
- Screening genomes for **biofilm formation, EPS production, antibiotic biosynthesis, and antagonistic traits**
- Prepared genomes and metadata for **NCBI submission**
- Contributed to **synthetic community design** using genomic and functional criteria

---

**🧩 Challenges & Solutions**

**Challenge 1:** Choosing appropriate assembly strategies for **200+ genomes** with varying read quality and genome complexity  
**Solution:** Implemented a **tiered assembly strategy**:

- Short-read–only assemblies using **Shovill/SPAdes** for high-quality Illumina datasets
- Long-read assemblies using **Flye** for structurally complex genomes
- **Hybrid assemblies (Illumina + Nanopore)** using **Unicycler** for isolates with fragmented short-read assemblies

---

**Challenge 2:** Resolving misassemblies, plasmids, and repetitive regions in hybrid assemblies  
**Solution:**

- Used **Bandage** for assembly graph visualization and manual validation
- Evaluated contiguity and structural accuracy using **assembly graphs and coverage profiles**
- Iteratively refined assemblies prior to downstream annotation and submission

---

**Challenge 3:** Handling metagenome complexity and distinguishing isolate genomes from MAGs  
**Solution:**

- Applied genome-resolved metagenomic workflows to separate **MAGs and isolate genomes**
- Cross-validated taxonomy and completeness using marker genes and coverage patterns
- Standardized QC across datasets before comparative analyses

---

**🛠 Methods & Tools**

\*_Data & Sequencing_

- Shotgun metagenomics (biofilm samples from UUV body and tail)
- Whole-genome sequencing of bacterial isolates
- Short-read, long-read, and **hybrid genome assemblies**
- Functional annotation of isolate genomes and MAGs
- Taxonomic classification and genome quality assessment

\*_Bioinformatics & Visualization_

- **Assembly & QC:** Flye, SPAdes, Shovill, Unicycler, NanoPlot
- **Assembly validation:** Bandage
- **Annotation:** Prokka, InterProScan, DRAM
- **Metagenomics:** Genome-resolved metagenomics workflows
- **Infrastructure:** High-performance computing (HPC)

\*_Languages & Workflow_

- Python
- Bash
- R
- Snakemake

---

**Publications**

- **Chaulagain D**, **John J**, Paul B, Harrington EG, McCarthy G, Sathe M, Shamabadi NS, Carter E, Leonhardt J, Nawaz MS, Suleman M, Campbell BJ, Karig DK.  
  _Genome assemblies of bacterial isolate collections from marine biofilm and water explore microbial diversity._  
  **Microbiology Resource Announcements** (submitted).

_(Additional manuscripts in preparation.)_

---

**🎤 Conferences & Talks**

- **Functional redundancy of marine synthetic biofilm communities under different environmental stresses**  
  Alisha M. Paul, **Jojy John**, Diptee Chaulagain, David Karig, Barbara J. Campbell  
  ASM Biofilms, Oregon, 2025

- **Genomic insights into antibiotic production and resistance in environmental bacteria**  
  Sophia Rudolph, **Jojy John**, Barbara J. Campbell  
  Regional ASM Meeting, South Carolina, 2025

- **Microbial biofouling interactions on Autonomous Navy Gliders: Insights from metagenome analysis**  
  **Jojy John**, David Karig, Barbara J. Campbell  
  DARPA Meeting, 2024

- **Isolation and identification of marine bacteria to enhance antifouling strategies in UUVs via a biofilm engineering approach**  
  ASM, Atlanta, 2024

- **Optimizing a stable and smooth biofilm-forming bacterial community to reduce drag on UUVs**  
  ICME, 2024

---

**🧑‍🔬 Collaborators / References**

**Dr. Barbara J. Campbell**  
Dean’s Distinguished Professor  
Department of Biological Sciences, Clemson University  
Email: bcampb7@clemson.edu

**Dr. David K. Karig**  
Associate Professor, Bioengineering  
Clemson University  
Email: dkarig@clemson.edu

**Dr. Diptee Chaulagain**  
Research Assistant Professor  
Department of Bioengineering, Clemson University  
Email: dchaula@clemson.edu

---
