---
layout: post
title: "Amplicon Week — Day 1: Introduction to 16S, ITS, 18S, 12S, and COI Metabarcoding"
date: 2025-12-08
description: "Day 1 of Amplicon sequencing and analyis."
comments: true
giscus_comments: true
featured: true
permalink: /blog/amplicon-day1/
series: amplicon
series_title: "Amplicon Week Series"
order: 1
---

**Welcome to Day 1 of my Amplicon Sequencing & Metabarcoding Week (Dec 8–13)!**
Throughout this week, I will walk through complete, reproducible workflows for 16S, ITS, 18S, 12S, and COI — from PCR primers to QIIME2 processing, visualization, functional prediction, and web-based tools.

---

#### 🔬 What Is Amplicon Sequencing?

Amplicon sequencing targets specific marker genes using PCR primers designed to capture biodiversity within a taxonomic group.
After sequencing, reads are filtered, denoised into ASVs (amplicon sequence variants) or OTUs, then assigned taxonomy.

Different marker genes illuminate different biological groups:

| **Marker Gene**     | **Target Group**   | **Typical Use Cases**                    |
| ------------------- | ------------------ | ---------------------------------------- |
| **16S rRNA**        | Bacteria + Archaea | Soil, water, host-associated microbiomes |
| **ITS (ITS1/ITS2)** | Fungi              | Soil fungome, rhizosphere, indoor fungi  |
| **18S rRNA**        | Eukaryotes         | Protists, plankton, marine ecology       |
| **12S rRNA**        | Vertebrates        | eDNA surveys, fish biodiversity          |
| **COI**             | Animals (Metazoa)  | DNA barcoding, insects, zooplankton      |

Each marker has unique primer sets, databases, taxonomic resolution, and best practices — which we will explore in the coming days.

#### 🧪 From DNA Extraction to Sequencing (Short Wet-Lab Overview)

Although this series focuses on computational workflows, here is a quick overview of upstream steps:

**DNA Extraction**

● Soil: Qiagen DNeasy PowerSoil / MagAttract kits

● Water: Sterivex filtration

● Host: Tissue, swabs, fecal samples

**PCR Amplification**

● 16S: 515F–806R (V4), 341F–805R (V3–V4), 27F–1492R (full-length)

● ITS: ITS1F–ITS2, ITS3–ITS4

● 18S: TAReuk454FWD1 / REV3 (Stoeck et al., 2010)

● 12S: MiFish primers (Miya et al., 2015)

● COI: mlCOIintF / jgHCO2198 (Leray et al., 2013)

**Sequencing**

● Illumina MiSeq or NovaSeq

● Typical read lengths: 2 × 250 bp or 2 × 150 bp

**Downstream Computational Workflow**

Denoising → Taxonomy → Filtering → Diversity → Functional Inference

#### 🔍 Deep Dive: 16S rRNA (Bacteria & Archaea)

16S contains conserved regions (primer binding) and hypervariable regions (V1–V9) which enable:

● Taxonomic profiling

● Community comparison

● Alpha/beta diversity analysis

● Environmental gradient interpretation

Because of its universality and broad databases (SILVA, Greengenes, GTDB), 16S is the backbone of microbial ecology, especially in soil and aquatic systems.

#### 🍄 ITS: The Fungal Marker

ITS evolves rapidly and provides species-level resolution for fungi.

● Common primers: ITS1F–ITS2, ITS3–ITS4

● Database: UNITE

ITS is essential for soil ecology, plant–microbe interactions, and indoor fungal surveys.

#### 🧬 18S rRNA for Microeukaryotes

18S captures a diverse range of eukaryotic lineages including:

● Protists

● Metazoa

● Databases: PR2, SILVA

This is the region I worked extensively on during my deep-sea project.
(Ramadoss, D., Ammanabrolu, B. S., Acharya, A., **John, J**., & Ingole, B. (2025). Deep-sea Life associated with sediments and polymetallic nodules from the Central Indian Ocean Basin: Insights from 18S metabarcoding. Deep Sea Research Part II: Topical Studies in Oceanography, 105487.)

#### 🐟 12S rRNA for Vertebrate eDNA

Used in environmental DNA (eDNA) surveys to detect:

● Fish

● Amphibians

● Marine mammals

● Primers: MiFish
● Databases: MitoFish, BOLD

#### 🐛 COI: The Universal Barcode Gene

COI provides excellent species-level resolution for animals and invertebrates.

● Primers: Leray et al. (2013)

● Databases: BOLD, MIDORI

COI is widely used in biodiversity studies, gut content metabarcoding, and soil invertebrate ecology.

#### 🛠 Popular Tools for Amplicon Data Analysis

1️⃣ QIIME2 (Recommended)

● DADA2/Deblur for denoising

● Taxonomic classification

● Diversity analyses

● High-quality visualizations

2️⃣ DADA2 (R package)

● Precise ASV inference

● Seamless integration with phyloseq

3️⃣ Mothur

● classic pipeline following MiSeq SOP

4️⃣ Microeco (R)

Visualization and ecological analysis

5️⃣ Web-Based Tools

● MicrobiomeAnalyst

● EZBioCloud

● Shiny-16S

#### Installation

For more details about installation of tools I am going to use in coming days; please visit my github project:
https://github.com/jojyjohn28/AmpliconWeek_2025

**Stay tuned for Day 2 (Dec 9): QIIME2 Setup, Databases & Classifier Training**

We will walk through importing data, visualizing reads, training classifiers for each marker gene, and denoising with DADA2.

![amplicon_day1](/assets/img/amplicon.png)
