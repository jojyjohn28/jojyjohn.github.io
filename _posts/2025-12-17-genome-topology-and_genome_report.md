---
layout: post
title: "Genome Topology and Genome Announcement Reports: A Practical, Reviewer-Safe Workflow"
date: 2025-12-17
description: "A step-by-step guide to answering NCBI genome topology questions and generating a complete, automated genome report table for Genome Resource Announcements."
comments: true
giscus_comments: true
featured: true
permalink: /blog/genome-topology-and-genome-report/
---

#### 🧬 Genome Topology and Genome Announcement Reports

Today, I’m stepping slightly back from the Size-Fractionated Microbiome Analysis series. With the winter break approaching, I wanted to wrap up a few pending tasks, including finalizing genome submissions to NCBI. We recently received the final queries and accession numbers for most of our genomes. So toaday I am documenting how to answer last few NCBI submission questions and how to prepare a complete genome summary table suitable for a Genome Resource Announcement manuscript.

**How to answer NCBI questions without over-claiming**

When submitting genomes to NCBI (GenBank / Genome / BioProject), two questions almost always appear and often cause confusion:

[A] Is this a draft assembly or does one sequence represent the chromosome?
[B] If this is a circular chromosome, are the ends of the sequence contiguous around the circle with no overlap and no gap?

These questions are not asking you to prove perfection — they are asking you to state clearly what you are claiming and what you are not.

This post documents a practical, conservative, and reviewer-safe workflow that I use to:

● Answer NCBI topology questions confidently

● Avoid over-claiming circularity

● Generate a final genome summary table suitable for Genome Resource Announcements

#### 🔑 Core principle (very important)

**Only assert circularity when the assembly method and validation steps truly support it.**

● Draft genomes and linear submissions are fully acceptable to NCBI.

● Over-claiming circularity is not

#### Step 1 — Determine whether a genome is draft or single-contig

For each genome FASTA:

```bash
grep -c "^>" genome.fa
```

I have many genomes and I copied all into one folder and used a batch script as follows

```bash
for f in *.fa; do
    echo "=== $f ==="
    grep -c "^>" "$f"
done
```

**Interpretation**
| Result | Meaning | NCBI Answer |
| --------- | -------------------- | ------------------- |
| >1 contig | Draft assembly | [A] Draft assembly |
| 1 contig | Candidate chromosome | Requires validation |
If a genome has multiple contigs, do not discuss circularity.

#### Step 2 — Validate circularity for single-contig genomes

This step applies only to genomes with exactly one contig.

**2a. Check assembler evidence (Flye / Unicycler)**

Inspect assembly_info.txt

Look for circular=true

If circularity is not explicitly stated, do not assume it

Illumina-only assemblies cannot reliably establish circularity.
As I used long read and hybrid assembly, I made a custom script to extract the inforamtion either from the assembly_info.txt or from the assembly. I also confirmed the results with step 2b

**2b. Check for overlap between contig ends**

NCBI requires no overlap and no gap at the contig ends.
**Extract first and last 1 kb**

```bash
seqkit subseq -r 1:1000 genome.fa > first1k.fa
seqkit subseq -r -999:-1 genome.fa > last1k.fa
```

You need seqkit tool for this and this can be insttaled as follows:
seqkit is a fast, lightweight toolkit for manipulating FASTA/FASTQ files. In this workflow, it is used to extract the first and last regions of contigs for circularity checks.

```bash
conda install -c bioconda seqkit
```

**Align ends**

```bash
nucmer --maxmatch first1k.fa last1k.fa -p ends
show-coords -rcl ends.delta
```

for this step you need the tool nucmer.
MUMmer is a whole-genome alignment package. Here, nucmer is used to align the start and end of contigs to detect overlapping duplicated regions, which is critical before claiming circularity.

```bash
conda install -c bioconda mummer
```

This installs nucmer and show-coords, which are used for end-to-end contig validation.

**Interpretation**
| Result | Meaning | Action |
| ---------------- | ---------------- | ---------------------------------- |
| Strong alignment | Overlap present | Must trim before claiming circular |
| No alignment | Not circularized | Treat as linear |

**If no alignments are shown (only headers):**

→ The genome is not circularized
→ Do not answer YES to question [B]
**🚫 Do not infer circularity for Illumina-only assemblies**

#### 🔁 Genome Resource Announcements

Although all of the analyses described below had been completed previously, the final genome FASTA files were subsequently renamed, filtered, and curated, and a subset of genomes was reassembled. Therefore, to generate a consistent and accurate summary table for the Genome Resource Announcement, I decided to define a clear work plan and rerun the relevant steps in a streamlined, reproducible workflow.

**Workflow overview**
| Tool | Purpose | Fields |
| ---------------- | ------------------- | --------------------------- |
| QUAST | Assembly statistics | Size, GC, contigs, N50 |
| CheckM2 | Quality metrics | Completeness, contamination |
| Prokka | Annotation | Protein-coding genes |
| Flye / Unicycler | Topology evidence | Circular / Linear |
| NUCmer | End validation | Overlap detection |

#### Step 1— Assembly statistics with QUAST

```bash
mkdir -p quast_out
quast.py *.fa -o quast_out --threads 16
```

I used batch script as follows

```bash
#!/bin/bash
#SBATCH --job-name=quast_all
#SBATCH --partition=camplab
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --mem=64G
#SBATCH --time=12:00:00
#SBATCH --output=quast_%j.out
#SBATCH --error=quast_%j.err

module load quast

GENOME_DIR="/project/dkarig/ecocoat/NCBI_oct22/fixed_genomes_dec11"
OUTDIR="/project/dkarig/ecocoat/NCBI_oct22/quast_dec11"

mkdir -p "$OUTDIR"

# Run QUAST on all genomes together
quast.py \
  --threads 16 \
  --min-contig 500 \
  -o "$OUTDIR" \
  "$GENOME_DIR"/*.fa

echo "QUAST completed at $(date)"
```

save this file as run_quast_all.sh and run as follows

```bash
chmod +x run_quast_all.sh
sbatch run_quast_all.sh
```

Final file is report.tsv:

● Genome size

● GC (%)

● number of Contigs

● N50

#### Step 2 — Completeness and contamination with CheckM2

CheckM was used to assess genome quality by estimating completeness and contamination based on lineage-specific single-copy marker genes. The lineage workflow automatically assigns each genome to an appropriate taxonomic lineage and reports standardized quality metrics, which were exported as a tab-delimited summary table and incorporated directly into the final genome summary table for the Genome Resource Announcement.

```bash
#!/bin/bash
#SBATCH --job-name=checkm_all
#SBATCH --partition=camplab
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=24
#SBATCH --mem=96G
#SBATCH --time=24:00:00
#SBATCH --output=checkm_%j.out
#SBATCH --error=checkm_%j.err

# Load CheckM module or activate conda env
conda activate checkm_env

GENOME_DIR="/project/dkarig/ecocoat/NCBI_oct22/all_genomes_dec15"
OUTDIR="/project/dkarig/ecocoat/NCBI_oct22/checkm_dec15"

mkdir -p "$OUTDIR"

# Run CheckM lineage workflow
checkm lineage_wf \
  -x fa \
  -t 24 \
  --reduced_tree \
  "$GENOME_DIR" \
  "$OUTDIR"

# Generate a clean summary table (TSV)
checkm qa \
  "$OUTDIR"/lineage.ms \
  "$OUTDIR" \
  -o 2 \
  -f "$OUTDIR/checkm_summary.tsv"

echo "CheckM completed at $(date)"
```

Save this as run_checkm_all.sh and run as follows

```bash
chmod +x run_checkm_all.sh
sbatch run_checkm_all.sh
```

**the key output file you need is checkm_summary.tsv**
This includes (per genome):

● Completeness (%)

● Contamination (%)

● Strain heterogeneity

● Marker lineage

#### Step 3 — Gene prediction with Prokka

This is for finding number of predicted genes and my strtegy is below

● Run Prokka once per genome (recommended; Prokka is per-genome by design)

● Store outputs in per-genome subfolders

● Parse Prokka outputs to create one summary TSV with:

● number of CDS (protein-coding genes)

● optionally rRNA, tRNA counts

```bash
#!/bin/bash
#SBATCH --job-name=prokka_all
#SBATCH --partition=camplab
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16
#SBATCH --mem=64G
#SBATCH --time=24:00:00
#SBATCH --output=prokka_%j.out
#SBATCH --error=prokka_%j.err

module load prokka

GENOME_DIR="/project/dkarig/ecocoat/NCBI_oct22/all_genomes_dec15"
OUTDIR="/project/dkarig/ecocoat/NCBI_oct22/prokka_dec15"

mkdir -p "$OUTDIR"

for fa in "$GENOME_DIR"/*.fa; do
    base=$(basename "$fa" .fa)
    echo "Running Prokka on $base"

    prokka \
      --outdir "$OUTDIR/$base" \
      --prefix "$base" \
      --cpus 16 \
      --force \
      "$fa"
done

echo "Prokka completed at $(date)"
```

save as run_prokka_all.sh and run as follows

```bash
chmod +x run_prokka_all.sh
sbatch run_prokka_all.sh
```

this will generate
For each genome (example ABV4_150):

```pgsql
prokka_dec15/ABV4_150/
├── ABV4_150.gff
├── ABV4_150.tbl
├── ABV4_150.txt   ← summary stats (VERY IMPORTANT)
├── ABV4_150.faa
├── ABV4_150.ffn
└── ABV4_150.log
```

The file \*.txt contains lines like:

```yaml
CDS: 4123
rRNA: 9
tRNA: 67
```

#### step 3.1 Generate a summary TSV for predicted genes

Script: prokka_summary.sh

This extracts CDS counts (protein-coding genes) for all genomes.

```bash
#!/bin/bash

PROKKA_DIR="/project/dkarig/ecocoat/NCBI_oct22/prokka_dec15"
OUT_TSV="prokka_gene_summary.tsv"

echo -e "Genome\tCDS\trRNA\ttRNA" > "$OUT_TSV"

for d in "$PROKKA_DIR"/*; do
    genome=$(basename "$d")
    txt="$d/$genome.txt"

    if [[ -f "$txt" ]]; then
        cds=$(grep "^CDS:" "$txt" | awk '{print $2}')
        rrna=$(grep "^rRNA:" "$txt" | awk '{print $2}')
        trna=$(grep "^tRNA:" "$txt" | awk '{print $2}')
        echo -e "$genome\t$cds\t$rrna\t$trna" >> "$OUT_TSV"
    fi
done

echo "Summary written to $OUT_TSV"
```

Run it:

```bash
chmod +x prokka_summary.sh
./prokka_summary.sh
```

#### Step 4 — Gene prediction with Prokka

Follow the steps from first section

#### 🧩 Step 5: Merge individual outputs into a final genome summary table

At this stage, all analyses have been completed independently, and the results are available as separate tabular files:

● prokka_out.tsv → predicted gene counts

● topology.tsv → genome topology (Circular / Linear / Undetermined)

● checkm_summary.tsv → completeness and contamination

● quast_report.tsv → genome size, GC content, contigs, N50

● NCBI_list_of_genomes.tsv → genome identifiers and accession numbers

**Each file contains a shared identifier column (Genome), which allows them to be merged programmatically into a single, consistent summary table.**

To avoid manual errors and ensure reproducibility, I used pandas in Python to merge all tables based on this common column.

🐍 Example: merging genome-level tables using pandas

```python
import pandas as pd

# Load individual result tables
prokka = pd.read_csv("prokka_out.tsv", sep="\t")
topology = pd.read_csv("topology.tsv", sep="\t")
checkm = pd.read_csv("checkm_summary.tsv", sep="\t")
quast = pd.read_csv("quast_report.tsv", sep="\t")
ncbi = pd.read_csv("NCBI_list_of_genomes.tsv", sep="\t")

# Ensure consistent column names
for df in [prokka, topology, checkm, quast, ncbi]:
    df.columns = df.columns.str.strip()

# Sequentially merge tables using the common 'Genome' column
final_table = (
    quast
    .merge(prokka, on="Genome", how="left")
    .merge(checkm, on="Genome", how="left")
    .merge(topology, on="Genome", how="left")
    .merge(ncbi, on="Genome", how="left")
)

# Save final table
final_table.to_csv("final_genome_summary_table.tsv", sep="\t", index=False)

print("Final genome summary table successfully created.")
```

This approach ensures that:

● All genome-level metadata are synchronized

● Missing values are handled transparently

● The workflow is fully reproducible

#### 📋 Final genome summary table format

The merged output is used directly as Table 1 in the Genome Resource Announcement manuscript.

Table 1. Genome assembly and annotation statistics
| Genome | Assembly type | Genome size (bp) | GC (%) | Coverage (×) | Contigs | N50 (bp) | Predicted genes | Completeness (%) | Contamination (%) | Topology | Accession |
| ------ | ------------- | ---------------- | ------ | ------------ | ------- | -------- | --------------- | ---------------- | ----------------- | -------- | --------- |
● Assembly type: Illumina / Hybrid / Long-read

● Topology: Circular / Linear / Undetermined

● Accession: “pending” during submission, updated post-acceptance

● genome statistics : N50, GC content, Completness, Contamination, Genome size, and Number of contigs

#### ✅ Take-home messages

● Do not over-claim genome circularity. Only assert circular topology when supported by long-read or hybrid assemblies and explicit validation.

● Draft and linear genomes are fully acceptable for NCBI submission and Genome Resource Announcements.

● Topology should be assigned conservatively using assembly evidence, not biological expectation.

● A single, reproducible summary table simplifies NCBI submissions and serves directly as a manuscript-ready resource.

● Clear documentation of methods and assumptions makes genome submissions reviewer-safe and future-proof.

This workflow has worked smoothly for large-scale genome submissions and Genome Resource Announcements.

The image is showing GC content of all the genomes.

![genomes_are_ready](/assets/img/gc.png)
