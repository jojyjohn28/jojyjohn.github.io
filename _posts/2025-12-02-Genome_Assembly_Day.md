---
layout: post
title: "Genome Assembly Day: Shovill for Illumina & Flye for Nanopore Reads"
date: 2025-12-02
description: "A practical walkthrough of assembling bacterial genomes using Shovill (Illumina short reads) and Flye (Nanopore long reads) on the Palmetto HPC cluster."
comments: true
giscus_comments: true
featured: true
permalink: /blog/genome-assembly-day/
categories: [genomics]
---

As I said in earlier post about NCBI submission (https://jojyjohn28.github.io/blog/NCBI-submission-cleaning/), I missed few genome assembly previously.
So Today, I am focusing on these missed assemblies.

**🧬 Whole-Genome Assembly Using Shovill & Flye**

Today’s work focused on reconstructing complete bacterial genomes using two complementary assembly strategies:

- **Shovill** for Illumina short reads
- **Flye** for Nanopore long reads

These assemblies are part of 200+ genomes of DARPA, which primarly focus on synthetic biofilm producing commuities.

---

## 🔍 Why Shovill (Instead of Running SPAdes Directly)

Shovill is a high-performance wrapper around SPAdes specifically optimized for bacterial WGS data.  
I chose it over raw SPAdes because Shovill provides:

✔ **Automatic adapter trimming**  
✔ **Read correction & subsampling** (reduces runtime)  
✔ **Optimized SPAdes parameters for bacteria**  
✔ **Cleaner, smaller output folders**  
✔ **Much faster and significantly lower memory usage**

For five bacterial genomes, Shovill saved hours of compute time and produced polished assemblies with minimal tweaking.

---

## ⚙️ Step 1 — Installing Shovill (Conda)

```bash
conda create -n shovill_env -c bioconda -c conda-forge shovill
conda activate shovill_env
```

## 🚀 Step 2 — Running Shovill on HPC (Batch Script)

Below is the script I used for assembling 5 isolates:

```bash
#!/bin/bash

READS_DIR="/project/dkarig/ecocoat/NCBI_oct22/need_biosamples/missing_assemblies"
OUTPUT_DIR="/project/dkarig/ecocoat/NCBI_oct22/need_biosamples/missing_assemblies/Shovil_assembly"
THREADS=32

mkdir -p "$OUTPUT_DIR"

for FORWARD_READ in "$READS_DIR"/*_R1.fastq.gz; do
    SAMPLE=$(basename "$FORWARD_READ" _R1.fastq.gz)
    REVERSE_READ="$READS_DIR/${SAMPLE}_R2.fastq.gz"

    if [[ -f "$REVERSE_READ" ]]; then
        echo "Processing $SAMPLE..."

        shovill \
            --outdir "$OUTPUT_DIR/$SAMPLE" \
            --R1 "$FORWARD_READ" \
            --R2 "$REVERSE_READ" \
            --cpus "$THREADS"

        echo "Assembly for $SAMPLE completed."
    else
        echo "Reverse read file missing for $SAMPLE. Skipping..."
    fi
done
```

save it as shovil.sh in the working directory (where raw sequences are saved)
direct the terminal to working directory

```bash
chmod +x shovil_new.sh
./shovil_new.sh
```

Shovill outputs:

● contigs.fa

● SPAdes log files

● assembly statistics

## 🧬 Long-Read Assembly Using Flye

For three Nanopore samples, I used Flye, a robust long-read assembler that works beautifully on HPC.

🔍 Why Flye (Instead of Epi2me Labs)

I prefer Flye because:

✔ Works entirely on HPC (no GUI required)

✔ No need for ONT’s desktop software

✔ Excellent for bacterial genomes

✔ Transparent logs & customizable parameters

✔ Supports polishing and repeat resolution

Epi2me is great for small standalone analyses, but Flye is far better for reproducible, scriptable, HPC-scale assembly.

## ⚙️ Step 1 — Installing Flye

```bash
git clone https://github.com/fenderglass/Flye.git
cd Flye
python setup.py install --user
```

## 🚀 Step 2 — Running Flye on an Interactive Node

I assembled each genome individually on my interactive node, as the number of genomes are less:

```bash
cd /home/jojyj/Flye
python /home/jojyj/Flye/bin/flye --nano-raw path to raw sequence --threads 32 --out-dir path to output folde
```

Flye outputs include:
● assembly.fasta

● assembly_graph.gfa

● assembly_info.txt

## 🧭 Summary

Today’s assembly workflow produced:

✔ Five high-quality Illumina assemblies (Shovill)

✔ Three Nanopore long-read assemblies (Flye)

✔ Clean contigs ready for downstream steps (QC, annotation, pangenome, etc.)

![Flye_ruuning](/assets/img/flye.png)
