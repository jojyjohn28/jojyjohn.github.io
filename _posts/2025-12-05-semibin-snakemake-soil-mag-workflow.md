---
layout: post
title: "Machine-learning MAG binning with SemiBin2 and Snakemake (soil metagenomes)"
date: 2025-12-05
description: "How I used SemiBin2 and a Snakemake workflow to recover high-quality MAGs from fragmented, high-diversity soil metagenomes."
comments: true
featured: true
giscus_comments: true
permalink: /blog/semibin-snakemake-soil-mag-workflow/
categories: [machine-learning]
tags: [machine-learning, semibin2, random-forest, binning, microbiome]
---

Today’s post is about one of my favorite combinations right now:  
**machine-learning–guided binning with SemiBin2** + **a fully automated Snakemake workflow** for recovering MAGs from **highly complex soil metagenomes**.

This is part of my broader goal of **machine-learning–guided multi-omics analysis**.

---

## 🧬 Why soil metagenomes are painful (and exciting)

I’m working with **soil metagenomes**, which are notoriously challenging:

- 1 g of soil can harbor **millions of bacterial, archaeal, and fungal cells**
- Community richness and evenness are extremely high
- Assemblies become **highly fragmented** with **low N50**
- Short contigs make binning much harder

After exploring different strategies, I moved to a **co-assembly** approach:

- Co-assembly with **MEGAHIT** of ~30 samples
- Producing a single `final.contigs.fa`

(I will post separate details on MEGAHIT parameters.)

---

## 🧱 Classical binners: what failed and what worked

I initially tested the usual multi-binner approaches:

### ❌ **CONCOCT**

Likely failed because:

- Assembly extremely large
- Could not consistently segment into 10 kb fragments
- Coverage patterns too noisy

### ❌ **MaxBin2**

Struggled due to:

- Too many contigs
- Sparse coverage across samples

### ⚠️ **MetaBAT2 (worked partially)**

- Produced ~500 bins
- Good, but clearly **under-represented soil diversity**
- Bins were fragmented and incomplete

To do better, I needed something that:

1. Scales to **large, fragmented assemblies**
2. Leverages **machine learning** rather than fixed heuristics

---

## 🤖 Enter SemiBin2: machine-learning–guided binning

SemiBin2 integrates:

1. **Sequence composition (k-mers)**
2. **Coverage profiles from multiple samples**
3. **Deep-learning-based embeddings**
4. Density-based clustering for final bins

🔗 Repo: https://github.com/BigDataBiology/SemiBin

For complex and diverse soil data, this ML approach provides a huge improvement.

I automated everything through **Snakemake** so the full workflow runs with one command.

---

## 🔁 My SemiBin2 + Snakemake workflow (high-level)

### **Inputs**

- Co-assembly: `final.contigs.fa`
- Paired-end reads: `sample_R1_nodup.fq`, `sample_R2_nodup.fq`

### **Workflow steps**

1. Build **Bowtie2 index**
2. Map each sample → **sorted BAM**
3. Create `bam_list.txt`
4. SemiBin2 **generate features**
5. SemiBin2 **train_self** (machine-learning step)
6. SemiBin2 **bin** (produces bins)
7. **Unzip** and convert `.fa.gz → .fa`
8. **MetaWRAP refinement**
9. **dRep dereplication**

All managed by Snakemake.

---

## 🚀 Running the workflow

Once the Snakefile is created:

```bash
snakemake -j 32
```

Snakemake will:

- Detect required jobs
- Build the DAG
- Run everything in parallel
- Restart only failed steps
- Ensure reproducibility

---

## 📈 Results from this workflow

### ⭐ **SemiBin2 initial output**

**6,563 bins** generated from the co-assembly.

### ⭐ **After refinement + dereplication**

Using:

- MetaWRAP bin refinement
- dRep dereplication
- ≥ **70% completeness**
- ≤ **5% contamination**

I recovered:

🔥 **~1,000 high-quality MAGs**

A major improvement over the **~500 bins from MetaBAT2** alone.

These high-quality MAGs will be used for:

- Taxonomy
- Functional annotation
- CAZyme profiling
- Energy metabolism markers
- MG–MTX expression integration
- Functional redundancy modeling (future post)

---

## 🔚 Wrap-up

This combined SemiBin2 + Snakemake workflow gave me:

- A reproducible, automated MAG recovery pipeline
- Better binning in extremely complex soil metagenomes
- A strong foundation for multi-omics integration
- ML-powered binning without manual tuning

---

## 💾 Full workflow on GitHub

I’ll maintain and update the full Snakemake pipeline here:

👉 https://github.com/jojyjohn28/semibin2-soil-mag-workflow
