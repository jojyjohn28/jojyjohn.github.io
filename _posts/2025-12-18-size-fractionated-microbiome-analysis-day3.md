---
layout: post
title: "Size-Fractionated Microbiome Analysis — Day 3: Species-Level Profiling with mOTUs"
date: 2025-12-18
description: "Day 3 of the size-fractionated microbiome series: mOTUs-based species-level profiling, batch processing, manual merging of profiles, and visualization."
comments: true
giscus_comments: true
featured: true
permalink: /blog/size-fractionated-microbiome-analysis-day3/
---

#### 🌊 Size-Fractionated Microbiome Analysis — Day 3

In Day 1 (https://jojyjohn28.github.io/blog/size-fractionated-microbiome-analysis-day1/), we focused on metagenome preprocessing, and in Day 2 (https://jojyjohn28.github.io/blog/size-fractionated-microbiome-analysis-day2/), we explored taxonomic profiling using Kaiju.

**Today, in Day 3, I introduce mOTUs v3, a marker-gene–based approach for species-level profiling of prokaryotes directly from short reads.**

This method is particularly powerful for read-based analyses, where genomes are not assembled or binned, and is equally applicable to DNA (total community) and RNA (active community) datasets.

#### 🧬 Primary tool: mOTUs v3

mOTUs profiles microbial communities using 40 universal single-copy marker genes, providing consistent, genome-informed species resolution.

**Why mOTUs?**

Uses conserved marker genes instead of whole-genome alignment

● Quantifies both known taxa (linked to reference genomes) and unknown mOTUs (derived from metagenomes)

● Produces directly comparable profiles across DNA and RNA datasets

● Outputs normalized relative abundances suitable for downstream statistics and visualization

This makes mOTUs ideal for size-fractionated microbiome studies, where consistent taxonomic resolution across multiple datasets is critical.

#### 🛠️ Installation and database setup

Install mOTUs

```bash
conda install -c bioconda motus
```

Verify installation:

```bash
motus --version
```

mOTUs ships with prebuilt marker-gene databases, so no separate database construction is required.
for more details refer : https://motu-tool.org/tutorial.html

#### 🧪 Running mOTUs on paired-end reads

Example command

```bash
motus profile \
  -f Control_23_S13_R1.fq \
  -r Control_23_S13_R2.fq \
  -o Control_23_S13_motus.tsv
```

General usage

```bash
# Total community (DNA)
motus profile \
  -f reads/dna/S1_R1.fq.gz \
  -r reads/dna/S1_R2.fq.gz \
  -o motus/S1_dna.motus.tsv

# Active community (RNA)
motus profile \
  -f reads/rna/S1_R1.fq.gz \
  -r reads/rna/S1_R2.fq.gz \
  -o motus/S1_rna.motus.tsv
```

The output table contains consensus taxonomy and relative abundance per sample.

#### 🚀 Batch processing on HPC (example)

Below is the batch script used to run mOTUs across Chesapeake Bay metagenomes.

```bash
#!/bin/bash
#SBATCH --job-name=motus_mg_CP
#SBATCH --nodes=1
#SBATCH --cpus-per-task=32
#SBATCH --mem=1500G
#SBATCH --time=72:00:00
#SBATCH -o motus_%A_%a.out
#SBATCH -e motus_%A_%a.err
#SBATCH -A camplab
#SBATCH -p camplab

set -euo pipefail

cd /project/bcampb7/camplab/AL_JJ_oct23

input_dir="/project/bcampb7/camplab/0_Raw_Data/Metagenome_estury/CB_DNA_trimmomatic"
output_dir="/project/bcampb7/camplab/AL_JJ_oct23/motus_output/cp"

mkdir -p "${output_dir}"/{profiles,logs}

source ~/.bashrc
conda activate /home/jojyj/.conda/envs/motus_env

for R1 in "${input_dir}"/*_R1.fastq; do
    sample=$(basename "${R1/_R1.fastq/}")
    R2="${input_dir}/${sample}_R2.fastq"

    motus profile \
        -f "$R1" \
        -r "$R2" \
        -o "${output_dir}/profiles/${sample}_motus.txt" \
        -n 32 \
        2> "${output_dir}/logs/${sample}.log"
done
```

#### 🧩 Merging mOTUs profiles across samples

Motus do have an built-in motus merge command (for me it failed for this dataset due to a error so instead of rerunning all my metegenomes;I used a robust bash-based merging approach, which is transparent and easy to repeat.)

**Step 1 — Navigate to profiles directory**

```bash
cd path/to/output/folder
```

**Step 2 — Fix headers (add sample name)**

```bash
for f in *_motus.txt; do
    sample=$(basename "$f" _motus.txt)
    awk -v s="$sample" 'NR==3{print "#consensus_taxonomy\t"s; next} {print}' "$f" > tmp && mv tmp "$f"
done
```

**Step 3 — Create clean tables for merging**

```bash
mkdir -p merged_clean

for f in *_motus.txt; do
    sample=$(basename "$f" _motus.txt)
    tail -n +3 "$f" > merged_clean/"$sample".tsv
done
```

**Step 4 — Merge all samples**

```bash
cd merged_clean

first=$(ls *.tsv | sort | head -n 1)
cut -f1 "$first" > clades.txt

for f in $(ls *.tsv | sort); do
    cut -f2 "$f" > "${f%.tsv}.ab"
done

paste clades.txt *.ab > merged_raw.tsv
sed '1s/#consensus_taxonomy/clade_name/' merged_raw.tsv > motus_merged_profiles.tsv
```

The resulting file (motus_merged_profiles.tsv) is analysis-ready.

##### 🧬 Generating the final order-level abundance table from mOTUs profiles

mOTUs outputs taxonomic assignments as consensus lineage strings, typically spanning multiple ranks (kingdom → species). For downstream ecological visualization (e.g., stacked bar plots and heatmaps), I summarized the data at the order level, which provides a good balance between resolution and interpretability for estuarine microbiomes.

**Overview of the approach**

1. Start from the merged mOTUs profile table (motus_merged_profiles.tsv)

2. Parse the consensus taxonomy lineage to extract the order-level annotation

3. Collapse species- and genus-level entries into their corresponding orders

4. Sum relative abundances across all mOTUs belonging to the same order

5. Produce a sample × order matrix for visualization and statistical analysis

**This approach ensures that:**

● Taxonomy is consistent across samples

● Unknown or poorly resolved taxa are handled transparently

● The resulting table is directly compatible with ggplot2, heatmaps, and multivariate analyses

#### 🧩 Step-by-step: Order-level table construction in R

I used "R" for this goal:

```r
library(tidyverse) #Load required packages
motus <- read_tsv("motus_merged_profiles.tsv") #Read merged mOTUs table
#The first column (clade_name) contains the consensus taxonomy lineage, followed by one column per sample.
motus_long <- motus %>%
  pivot_longer(
    cols = -clade_name,
    names_to = "Sample",
    values_to = "Abundance"
  )
#Reshape to long format
#Extract order-level taxonomy from lineage strings
#mOTUs taxonomy strings typically follow this structure:
#k__Bacteria|p__Proteobacteria|c__Gammaproteobacteria|o__Alteromonadales|...
#We extract the o__ component and clean it for plotting.
motus_long <- motus_long %>%
  mutate(
    order = str_extract(clade_name, "o__[^|]+"),
    order = str_remove(order, "^o__"),
    order = if_else(is.na(order), "Unclassified", order)
  )
#Summarize abundances at the order level
order_level <- motus_long %>%
  group_by(order, Sample) %>%
  summarise(
    Abundance = sum(Abundance),
    .groups = "drop"
  )
#At this stage, each sample has a relative abundance profile summarized by order.
```

This table was used as the final order-level input for:

● Stacked bar plots (Day 3)

● Heatmaps and multivariate analyses (later days)

● network and co-occurance modelling in coming months

#### 📊 Visualization in R

Stacked bar plots (order level, bay-wise)

```r
library(tidyverse)
library(readxl)

df <- read_excel(
  "/home/jojy-john/Jojy_Research_Sync/Fl_vs_Pa/total_and_active_community/order_level_summed.xlsx"
)

df_long <- df %>%
  pivot_longer(
    cols = -order,
    names_to = "Sample",
    values_to = "Abundance"
  ) %>%
  mutate(
    Bay = case_when(
      str_detect(Sample, "^CP") ~ "Chesapeake",
      str_detect(Sample, "^DE") ~ "Delaware",
      TRUE ~ "Unknown"
    ),
    order = str_trim(order),
    order = ifelse(order %in% c("Other", "Others"), "Other", order)
  )

top_orders <- df_long %>%
  filter(order != "Other") %>%
  group_by(order) %>%
  summarise(total = sum(Abundance), .groups = "drop") %>%
  arrange(desc(total)) %>%
  slice_head(n = 15) %>%
  pull(order)

df_long2 <- df_long %>%
  mutate(order2 = ifelse(order %in% top_orders, order, "Other")) %>%
  group_by(order2, Sample, Bay) %>%
  summarise(Abundance = sum(Abundance), .groups = "drop")

ggplot(df_long2, aes(x = Sample, y = Abundance, fill = order2)) +
  geom_bar(stat = "identity", position = "fill") +
  facet_wrap(~Bay, scales = "free_x") +
  labs(
    x = "Samples",
    y = "Relative abundance",
    fill = "Order"
  ) +
  theme_minimal(base_size = 13) +
  theme(
    axis.text.x = element_text(angle = 60, hjust = 1, face = "bold"),
    strip.text = element_text(face = "bold"),
    legend.title = element_text(face = "bold")
  )
```

#### ✅ Strengths of mOTUs in this study

● Species-level resolution without MAGs

● Consistent taxonomy across DNA and RNA

● Robust handling of unknown taxa

● Clean, normalized outputs for statistical analysis

#### 🔄 Kaiju vs mOTUs: Consistency across methods

To assess methodological consistency, I compared Kaiju (protein-level classification) and mOTUs (marker-gene–based profiling) outputs at higher taxonomic ranks. **Despite relying on fundamentally different approaches, both methods recovered highly similar dominant taxa across samples, particularly when comparisons were made at the order level.**

![total_community_motu](/assets/img/motu.png)
