---
layout: post
title: "Who Is Actually Active? Uncovering Active Viral Communities from Metatranscriptomes"
date: 2026-05-19
description: "A virus detected in a metagenome might be dormant, dead, or relic DNA. A virus detected in a metatranscriptome is alive, active, and doing something right now. This final post in the viromics series maps metatranscriptomic reads to viral genomes, quantifies active viruses, identifies expressed auxiliary metabolic genes, and shows what viral activity reveals about ecosystem function that DNA alone never could."
comments: true
giscus_comments: true
featured: true
series: viromics
order: 6
permalink: /blog/viromics-day6-active-viral-communities/
tags:
  [
    viromics,
    metatranscriptomics,
    active-viruses,
    AMG-expression,
    Bowtie2,
    featureCounts,
    CoverM,
    TPM,
    RPKM,
    viral-activity,
    RNA,
    DRAM-v,
    KEGG,
    environmental-microbiome,
    phage,
    bioinformatics,
    beginners,
  ]
---

## The question everything has been building toward

_🧬 𝐷𝑎𝑦 83 𝑜𝑓 𝐷𝑎𝑖𝑙𝑦 𝐵𝑖𝑜𝑖𝑛𝑓𝑜𝑟𝑚𝑎𝑡𝑖𝑐𝑠 𝑓𝑟𝑜𝑚 𝐽𝑜𝑗𝑦'𝑠 desk_

Over five posts we have built a comprehensive picture of the viral community in our environmental samples. We identified viral genomes, clustered them into vOTUs, assigned taxonomy, measured DNA-based abundance, identified auxiliary metabolic genes, predicted hosts, determined lifestyle, and placed every virus in a protein-sharing network alongside the known viral universe.

All of that work was built on DNA. And DNA has a fundamental limitation: **presence is not the same as activity.**

A viral genome detected in a metagenome might be:

- An actively replicating phage in the middle of a lytic infection
- A dormant prophage integrated silently into a host chromosome
- A viral particle that infected a cell three weeks ago and has not replicated since
- Relic DNA from a lysed cell that persisted in the environment long after the virus stopped replicating
- Even contamination from a related environment

None of these scenarios looks different at the DNA level. But they are profoundly different ecologically. A dormant prophage contributes nothing to nutrient cycling or microbial mortality right now. An actively replicating cyanophage in the middle of a bloom is responsible for a measurable fraction of primary productivity loss in that water column at that moment.

**Metatranscriptomics — sequencing the RNA from an environmental sample — gives us the missing layer.** Viral RNA represents active transcription, which means active infection, active replication, and active interaction with the host cell. This post walks through how to generate and interpret that signal.

---

## Part 1 — DNA vs RNA: the central conceptual framework

Before any commands, it is worth spending time on the conceptual foundation, because it changes how you interpret everything that follows.

### The metagenome tells you potential

A metagenome is a snapshot of all the DNA present in a sample at the moment of collection. It tells you:

- Which viruses _could_ be active (their genomes are present)
- Which AMGs _could_ be expressed (the genes exist)
- Which hosts _could_ be infected (the phages are there)

But DNA is stable. A viral genome can persist in an environment for days, weeks, or even months after active replication has stopped. Environmental DNA degrades slowly, and particles can remain intact long after the virus inside them has lost infectivity.

### The metatranscriptome tells you reality

A metatranscriptome sequences the RNA present at the moment of collection. RNA is unstable — it degrades in minutes to hours without active transcription to maintain it. When you detect viral RNA in an environmental sample, you are detecting something that was transcribed **very recently**, almost certainly within the past few hours.

Viral RNA expression in the environment signals:

- **Active lytic infection** — a phage is inside a host cell right now, transcribing its genes to make new particles
- **Active lysogeny maintenance** — a temperate phage is expressing its repressor and lysogeny maintenance genes while integrated in the host chromosome
- **Active AMG expression** — a phage is expressing host metabolic genes during infection, directly manipulating the host's metabolism

This is why metatranscriptomics is considered the most direct window we have into what viruses are _doing_ in an ecosystem, as opposed to what they are _capable of doing_.

### The comparison that matters

| Layer                   | Data type                  | What it tells you                                          | Limitation                                                   |
| ----------------------- | -------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------ |
| Metagenome (DNA)        | Viral genome sequences     | Viral presence, potential function, diversity              | Cannot distinguish active from dormant or relic              |
| Metatranscriptome (RNA) | Viral transcript sequences | Active transcription, expressed genes, real-time infection | Captures only the moment of collection; RNA degrades rapidly |

The most powerful analyses in environmental viromics compare these two layers directly — looking for viruses that are present in DNA but absent from RNA (dormant/relic), and viruses that are abundant in RNA but low in DNA (highly active per genome copy, possibly amplifying rapidly).

### The dormant vs active dichotomy

This comparison can be formalised into a simple conceptual panel that is worth keeping in mind throughout this analysis:

```
METAGENOME (DNA)          METATRANSCRIPTOME (RNA)
─────────────────         ──────────────────────────
Viral presence        →   Viral activity
Potential function    →   Expressed function
Who is there          →   Who is doing something
Community composition →   Community function
Static snapshot       →   Dynamic real-time signal
```

A virus that appears in both DNA and RNA is **actively replicating**.
A virus that appears only in DNA is **present but dormant or relic**.
A virus that appears only in RNA is **rare by copy number but highly transcriptionally active** — potentially in the middle of a rapid lytic burst.

---

## Part 2 — Preparing the viral reference database

Before you can map metatranscriptomic reads to viral genomes, you need a reference database of viral sequences. This is your `vOTUs.fa` from Day 1, but it is worth thinking about what exactly you want to include.

### What to include in the reference

**At minimum:**

- `vOTUs.fa` — your representative viral genomes from Days 1–5

**Ideally add:**

- `checkv_output/proviruses.fna` — prophage sequences trimmed by CheckV; these are the integrated viruses most likely to show lysogeny maintenance transcription
- Any curated viral contigs from publicly available environmental datasets that are relevant to your system

```bash
# Combine vOTUs and proviruses into a single viral reference
cat vOTUs.fa checkv_output/proviruses.fna > viral_reference_db.fa

# Check the combined size
grep -c '^>' viral_reference_db.fa
seqkit stats viral_reference_db.fa
```

### Preparing gene annotations for featureCounts

featureCounts needs to know where each gene is on each viral contig so it can count reads per gene rather than per whole genome. Use the Prodigal GFF output from Day 5 (or regenerate it):

```bash
# If not already done in Day 5
module load prodigal/2.6.3

prodigal \
    -i viral_reference_db.fa \
    -a viral_reference_db.faa \
    -p meta \
    -f gff \
    -o viral_reference_db.gff \
    -q

echo "Genes predicted: $(grep -v '^#' viral_reference_db.gff | wc -l)"
```

---

## Part 3 — Quality trimming metatranscriptomic reads

Metatranscriptomic reads require the same quality trimming as metagenomic reads, but with one additional step: **rRNA depletion or filtering**.

Environmental RNA samples contain large amounts of ribosomal RNA (rRNA) from bacteria and archaea. If your library preparation did not include rRNA depletion (e.g. using Ribo-Zero kits), your data may be 80–95% rRNA reads that will not map to viral genomes and will dilute your signal. Check your raw data first:

```bash
# Count reads
echo "Total reads: $(zcat sample_RNA_R1.fastq.gz | wc -l | awk '{print $1/4}')"
```

### Quality trimming with fastp

```bash
fastp \
    -i sample_RNA_R1.fastq.gz \
    -I sample_RNA_R2.fastq.gz \
    -o sample_RNA_R1_trimmed.fastq.gz \
    -O sample_RNA_R2_trimmed.fastq.gz \
    --thread 16 \
    --detect_adapter_for_pe \
    --qualified_quality_phred 20 \
    --length_required 50 \
    --json sample_RNA_fastp.json \
    --html sample_RNA_fastp.html
```

### Optional: in silico rRNA filtering with SortMeRNA

If your library was not rRNA-depleted in the lab, filter computationally:

```bash
conda activate /your/tools/dir/sortmerna_env

sortmerna \
    --ref /path/to/sortmerna_dbs/smr_v4.3_fast_db.fasta \
    --reads sample_RNA_R1_trimmed.fastq.gz \
    --reads sample_RNA_R2_trimmed.fastq.gz \
    --workdir sortmerna_workdir \
    --aligned  sample_rRNA \
    --other    sample_non_rRNA \
    --threads 16 \
    --paired-in \
    --fastx

echo "Non-rRNA reads kept: $(zcat sample_non_rRNA_fwd.fastq.gz | wc -l | awk '{print $1/4}')"
```

The `sample_non_rRNA_fwd.fastq.gz` and `sample_non_rRNA_rev.fastq.gz` files are what you use for mapping.

---

## Part 4 — Building the Bowtie2 index and mapping RNA reads

### Build the Bowtie2 index

This is the same process as Day 2 abundance estimation, but now the index is built from `viral_reference_db.fa` (which includes proviruses):

```bash
conda activate /your/tools/dir/bowtie2_coverm_env

mkdir -p bowtie2_RNA_index

bowtie2-build \
    viral_reference_db.fa \
    bowtie2_RNA_index/viral_reference \
    --threads 16

echo "Index built: bowtie2_RNA_index/viral_reference"
```

### Map RNA reads to viral genomes

The mapping parameters for RNA reads differ slightly from DNA. We use `--very-sensitive` and allow a higher number of mismatches to accommodate RNA editing and minor sequence variation between the reference and transcribed sequence:

```bash
conda activate /your/tools/dir/bowtie2_coverm_env

SAMPLE="Control_23_RNA"
R1="${SAMPLE}_R1_trimmed.fastq.gz"
R2="${SAMPLE}_R2_trimmed.fastq.gz"

bowtie2 \
    -x bowtie2_RNA_index/viral_reference \
    -1 "$R1" \
    -2 "$R2" \
    --end-to-end \
    --very-sensitive \
    --no-unal \
    -p 16 \
    2> "${SAMPLE}_RNA_mapping.log" | \
    samtools sort -@ 4 -o "${SAMPLE}_RNA_sorted.bam"

samtools index "${SAMPLE}_RNA_sorted.bam"

# Check mapping rate
grep "overall alignment rate" "${SAMPLE}_RNA_mapping.log"
```

**Expected mapping rates to viral references from metatranscriptomes:**

Viral RNA is a small fraction of total environmental RNA even after rRNA filtering. Typical mapping rates are:

- 0.1–2% in bulk metatranscriptomes without rRNA depletion
- 1–15% in rRNA-depleted metatranscriptomes
- Up to 30–50% in samples collected during active viral blooms

Low mapping rates are normal and do not indicate a problem with the analysis.

---

## Part 5 — Quantifying active viruses with CoverM

CoverM calculates genome-level coverage metrics from the RNA-mapping BAM files, giving you abundance estimates for each viral genome in each sample at the transcriptional level.

```bash
conda activate /your/tools/dir/bowtie2_coverm_env

coverm genome \
    --bam-files *_RNA_sorted.bam \
    --genome-fasta-files viral_reference_db.fa \
    -m mean relative_abundance covered_fraction rpkm \
    --min-covered-fraction 0 \
    -t 16 \
    --output-file coverm_RNA_abundance.tsv
```

Note the addition of `rpkm` to the metrics — this normalises by both sequencing depth and genome length, which is important for comparing transcript abundance across viruses of different sizes.

### Applying the coverage filter to RNA data

Apply the same 75% covered-fraction filter as in Day 2, but note that this filter may be more stringent than needed for RNA data. Active viruses during lytic infection can show uneven coverage (high coverage over structural genes, lower over regulatory regions). Consider lowering the threshold to 50% for RNA:

```bash
python scripts/filter_coverm_abundance.py \
    --input  coverm_RNA_abundance.tsv \
    --output coverm_RNA_filtered.tsv \
    --threshold 0.50    # 50% for RNA vs 75% for DNA
```

---

## Part 6 — Gene-level quantification with featureCounts

While CoverM gives you genome-level activity metrics, **featureCounts** gives you gene-level read counts — essential for identifying which specific genes are being transcribed, particularly for AMG expression analysis.

### Install featureCounts (part of Subread)

```bash
module load anaconda3/2023.09-0

conda create \
    -p /your/tools/dir/subread_env \
    -c conda-forge -c bioconda \
    subread \
    -y

conda activate /your/tools/dir/subread_env
featureCounts -v
```

### Run featureCounts

```bash
conda activate /your/tools/dir/subread_env

featureCounts \
    -T 16 \
    -p \
    --countReadPairs \
    -a viral_reference_db.gff \
    -F GFF \
    -t CDS \
    -g ID \
    -o featurecounts_RNA_raw.tsv \
    *_RNA_sorted.bam \
    2> featurecounts.log
```

**Key parameters:**

| Parameter          | Meaning                                                 |
| ------------------ | ------------------------------------------------------- |
| `-p`               | paired-end mode                                         |
| `--countReadPairs` | count fragment pairs rather than individual reads       |
| `-a`               | annotation file (GFF from Prodigal)                     |
| `-F GFF`           | annotation format                                       |
| `-t CDS`           | feature type to count (Prodigal labels all ORFs as CDS) |
| `-g ID`            | attribute to use as gene ID                             |

The output `featurecounts_RNA_raw.tsv` has one row per gene and one column per sample, with raw read counts.

### Normalise to TPM

Raw read counts are not comparable between samples with different sequencing depths. **Transcripts Per Million (TPM)** normalises for both sequencing depth and gene length, making values comparable across samples and between genes of different sizes.

```python
import pandas as pd
import numpy as np

# Load featureCounts output (skip the first comment line)
fc = pd.read_csv("featurecounts_RNA_raw.tsv", sep="\t", comment="#")
fc.columns = fc.columns.str.strip()

# featureCounts output columns:
# Geneid, Chr, Start, End, Strand, Length, sample1.bam, sample2.bam ...
gene_lengths = fc["Length"]
count_cols   = [c for c in fc.columns if c.endswith(".bam")]

# TPM normalisation
# Step 1: RPK = reads per kilobase
rpk = fc[count_cols].divide(gene_lengths / 1000, axis=0)

# Step 2: per-million scaling factor
scaling = rpk.sum(axis=0) / 1e6

# Step 3: TPM
tpm = rpk.divide(scaling, axis=1)

# Build clean output
tpm_out = pd.DataFrame({
    "gene_id": fc["Geneid"],
    "genome":  fc["Chr"],
    "length":  gene_lengths
})
# Clean column names (remove .bam suffix)
tpm.columns = [c.replace("_RNA_sorted.bam", "").replace("_sorted.bam", "")
               for c in tpm.columns]
tpm_out = pd.concat([tpm_out, tpm], axis=1)
tpm_out.to_csv("gene_expression_TPM.tsv", sep="\t", index=False)
print(f"TPM table: {tpm_out.shape[0]:,} genes × {tpm_out.shape[1]} columns")
```

---

## Part 7 — Identifying actively transcribed viruses

With genome-level CoverM RNA data and gene-level TPM values, you can now define which viruses are **active** in your samples.

### Defining active viruses

A practical definition for an actively transcribed virus:

1. **Covered fraction ≥ 50%** of the viral genome has RNA coverage (not just a few conserved genes attracting non-specific reads)
2. **Mean RPKM > threshold** — suggest using the 75th percentile of all non-zero RPKM values in a sample as a dynamic threshold, or a fixed value like RPKM > 1

```python
import pandas as pd

rna = pd.read_csv("coverm_RNA_filtered.tsv", sep="\t", index_col=0)
dna = pd.read_csv("results/coverm_abundance_filtered.tsv", sep="\t", index_col=0)

# Extract RPKM columns from RNA table
rpkm_cols = [c for c in rna.columns if "RPKM" in c]
cf_cols_rna = [c for c in rna.columns if "Covered Fraction" in c]

# Mark viruses as active: RPKM > 1 AND covered_fraction >= 0.5
active_mask = pd.DataFrame(index=rna.index)
for i, rpkm_col in enumerate(rpkm_cols):
    sample = rpkm_col.replace(" RPKM", "").strip()
    cf_col = [c for c in cf_cols_rna if sample in c]
    if cf_col:
        active_mask[sample] = (rna[rpkm_col] > 1) & (rna[cf_col[0]] >= 0.5)
    else:
        active_mask[sample] = rna[rpkm_col] > 1

active_mask["n_active_samples"] = active_mask.sum(axis=1)
active_mask["is_any_active"]    = active_mask["n_active_samples"] > 0

n_active = active_mask["is_any_active"].sum()
n_total  = len(active_mask)
print(f"Active in at least one sample: {n_active} / {n_total} vOTUs ({n_active/n_total*100:.1f}%)")

active_mask.to_csv("results/active_virus_calls.tsv", sep="\t")
```

### DNA vs RNA: dormant vs active comparison

This is the most powerful comparison in the entire pipeline:

```python
import pandas as pd

# DNA relative abundance (presence)
dna_ra = pd.read_csv("results/coverm_abundance_filtered.tsv", sep="\t", index_col=0)
dna_ra_cols = [c for c in dna_ra.columns if "Relative Abundance" in c]

# RNA relative abundance (activity)
rna_ra = pd.read_csv("coverm_RNA_filtered.tsv", sep="\t", index_col=0)
rna_ra_cols = [c for c in rna_ra.columns if "Relative Abundance" in c]

# Mean across samples
dna_mean = dna_ra[dna_ra_cols].mean(axis=1).rename("mean_DNA_abundance")
rna_mean = rna_ra[rna_ra_cols].mean(axis=1).rename("mean_RNA_abundance")

comparison = pd.concat([dna_mean, rna_mean], axis=1).fillna(0)
comparison.index.name = "vOTU"

# Classify each vOTU
def classify_activity(row):
    dna = row["mean_DNA_abundance"]
    rna = row["mean_RNA_abundance"]
    if dna > 0 and rna > 0:
        return "Active"
    elif dna > 0 and rna == 0:
        return "Dormant_or_Relic"
    elif dna == 0 and rna > 0:
        return "High_activity_low_copy"
    else:
        return "Absent"

comparison["activity_class"] = comparison.apply(classify_activity, axis=1)
comparison.reset_index().to_csv("results/dna_rna_comparison.tsv", sep="\t", index=False)

print("\nActivity classification:")
print(comparison["activity_class"].value_counts().to_string())
```

---

## Part 8 — AMG expression analysis

This is the most ecologically powerful result you can generate from metatranscriptomic data. Not just: _viruses carry genes for sulfur metabolism._ But: _those genes are being actively expressed right now, in this sample, during infection._

### Linking gene-level TPM to AMG annotations

Your DRAM-v AMG table from Day 3 contains gene IDs and KEGG annotations. Your TPM table from featureCounts contains gene IDs and expression values. Join them:

```python
import pandas as pd

tpm  = pd.read_csv("gene_expression_TPM.tsv", sep="\t")
amg  = pd.read_csv("results/high_confidence_amgs.tsv", sep="\t")

# Normalise gene IDs — DRAM-v uses scaffold||score_geneN format
# featureCounts uses Prodigal's ID format
amg["gene_id_clean"] = (
    amg["gene_id"]
    .astype(str)
    .str.replace(r"\|\|.*", "", regex=True)
)
tpm["gene_id_clean"] = tpm["gene_id"].astype(str).str.strip()

# Merge
amg_expr = amg.merge(
    tpm,
    on="gene_id_clean",
    how="left",
    indicator=True
)

matched = (amg_expr["_merge"] == "both").sum()
print(f"AMG genes with expression data: {matched} / {len(amg_expr)}")
amg_expr.drop(columns=["_merge"], inplace=True)
amg_expr.to_csv("results/amg_expression.tsv", sep="\t", index=False)
```

### Which AMG categories are most expressed?

```python
import pandas as pd

amg_expr = pd.read_csv("results/amg_expression.tsv", sep="\t")

# Get TPM sample columns
sample_cols = [c for c in amg_expr.columns
               if c not in {"gene_id", "gene_id_clean", "genome", "length",
                             "scaffold", "kegg_hit", "ko_id", "metabolic_category",
                             "amg_flags", "assigned_family", "vOTU"}
               and "TPM" not in c]

# Identify actual sample columns (numeric)
import pandas.api.types as pat
tpm_cols = [c for c in amg_expr.columns
            if c not in {"gene_id", "gene_id_clean", "genome", "length",
                         "scaffold", "kegg_hit", "ko_id", "metabolic_category",
                         "amg_flags", "assigned_family", "vOTU"}
            and pat.is_numeric_dtype(amg_expr[c])]

kegg_col = next((c for c in ["kegg_hit", "ko_id"] if c in amg_expr.columns), None)
cat_col  = "metabolic_category" if "metabolic_category" in amg_expr.columns else None

if kegg_col and tpm_cols:
    amg_expr["mean_TPM"] = amg_expr[tpm_cols].mean(axis=1)

    # Top expressed AMG functions
    top_amg = (
        amg_expr[amg_expr["mean_TPM"] > 0]
        .groupby(kegg_col)["mean_TPM"]
        .agg(["mean", "sum", "count"])
        .rename(columns={"mean": "mean_TPM", "sum": "total_TPM", "count": "n_genes"})
        .sort_values("total_TPM", ascending=False)
        .head(20)
    )
    top_amg.to_csv("results/top_expressed_amgs.tsv", sep="\t")

    print("\nTop 20 expressed AMG KEGG functions:")
    print(f"  {'Function':<50} {'Mean TPM':>10}  {'Total TPM':>10}")
    print("  " + "-" * 75)
    for func, row in top_amg.iterrows():
        print(f"  {str(func)[:50]:<50} {row['mean_TPM']:>10.2f}  {row['total_TPM']:>10.2f}")
```

### Examples of biologically meaningful AMG expression

The ecological significance of expressed AMGs depends entirely on your environment. Here are examples of what active AMG expression tells you:

**Photosystem II genes expressed by cyanophages**
If your metatranscriptome shows high TPM for _psbA_ or _psbD_ in viral genes, cyanophages are actively infecting photosynthetic bacteria and expressing these genes to sustain photosynthesis during lytic infection. This is a direct contribution of viral activity to carbon fixation rates — the virus is keeping the photosynthetic machinery running while it hijacks the cell.

**Sulfur metabolism genes expressed in estuary viruses**
High TPM for _dsrA_, _dsrC_, or _sat_ in viral genes from an estuarine sample means viruses are actively infecting sulfate-reducing or sulfur-oxidising bacteria and expressing these pathway genes. During seasons of high organic matter loading, this signal can indicate that viral activity is accelerating sulfur cycling beyond what the bacterial community alone would sustain.

**Carbon metabolism genes expressed during high-productivity periods**
Expressed _zwf_ (glucose-6-phosphate dehydrogenase) or _pykA_ (pyruvate kinase) in viral genes during bloom conditions means phages are boosting glycolytic flux in their hosts to provide more energy and nucleotide precursors for viral replication. This is virus-directed metabolic reprogramming in real time.

---

## Part 9 — Ecological interpretation: connecting activity to environment

### Seasonal viral activity

If your dataset spans multiple time points or seasons, comparing RNA-based activity across time reveals patterns invisible to DNA:

```python
import pandas as pd

# Load RNA abundance with sample metadata
rna = pd.read_csv("coverm_RNA_filtered.tsv", sep="\t", index_col=0)
meta = pd.read_csv("sample_metadata.tsv", sep="\t")  # season, date, temp, etc.

rna_ra_cols = [c for c in rna.columns if "Relative Abundance" in c]

# Transpose to samples × vOTUs
rna_T = rna[rna_ra_cols].T.copy()
rna_T.index = rna_T.index.str.replace(" Relative Abundance (%)", "").str.strip()
rna_T.index.name = "sample"
rna_T = rna_T.reset_index().merge(meta, on="sample", how="left")

# Average activity by season
if "season" in rna_T.columns:
    seasonal = rna_T.groupby("season")[rna_T.columns[1:]].mean()
    seasonal.to_csv("results/seasonal_viral_activity.tsv", sep="\t")
    print("Seasonal activity table written")
```

### Free-living vs particle-associated viral activity

In many environmental viromics studies, samples are size-fractionated into:

- **Free-living (FL)** fraction: 0.2–2 µm filter — free virions and small bacteria
- **Particle-associated (PA)** fraction: > 2 µm filter — particles, aggregates, larger cells

Comparing viral RNA between fractions reveals whether active viral infections are associated with particles (perhaps indicating active lytic infections of particle-associated bacteria) or distributed freely (perhaps indicating active replication during dispersal).

### Connecting to the master ecological table

Merge the RNA activity data into your master table from Day 4:

```python
import pandas as pd

master    = pd.read_csv("results/master_ecological_table.tsv", sep="\t")
active    = pd.read_csv("results/active_virus_calls.tsv", sep="\t")
dna_rna   = pd.read_csv("results/dna_rna_comparison.tsv", sep="\t")
amg_expr  = pd.read_csv("results/amg_expression.tsv",     sep="\t")

# AMG expression summary per vOTU
tpm_cols = [c for c in amg_expr.columns if amg_expr[c].dtype in ["float64", "int64"]
            and c not in {"length"}]
if tpm_cols:
    amg_expr["mean_AMG_TPM"] = amg_expr[tpm_cols].mean(axis=1)
    amg_votu_expr = (
        amg_expr.groupby("vOTU")["mean_AMG_TPM"]
        .agg(["mean", "max", "count"])
        .rename(columns={"mean": "mean_amg_tpm",
                         "max":  "max_amg_tpm",
                         "count": "n_expressed_amgs"})
        .reset_index()
    )
    master = master.merge(amg_votu_expr, on="vOTU", how="left")

master = master.merge(
    active[["vOTU", "n_active_samples", "is_any_active"]],
    on="vOTU", how="left"
)
master = master.merge(
    dna_rna[["vOTU", "mean_RNA_abundance", "activity_class"]],
    on="vOTU", how="left"
)

master.to_csv("results/master_ecological_table_final.tsv", sep="\t", index=False)
print(f"Final master table: {master.shape[0]} vOTUs × {master.shape[1]} columns")

# Summary
print("\nActivity class breakdown:")
if "activity_class" in master.columns:
    print(master["activity_class"].value_counts().to_string())
```

---

## Part 10 — Visualisation ideas

### Heatmap of top active vOTUs × samples

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

rna = pd.read_csv("coverm_RNA_filtered.tsv", sep="\t", index_col=0)
rna_ra_cols = [c for c in rna.columns if "Relative Abundance" in c]

# Top 40 most active vOTUs by max RNA abundance
top40 = rna[rna_ra_cols].max(axis=1).nlargest(40).index
rna_top = rna.loc[top40, rna_ra_cols].copy()
rna_top.columns = [c.replace(" Relative Abundance (%)", "").strip()
                   for c in rna_top.columns]

fig, ax = plt.subplots(figsize=(12, 14))
sns.heatmap(
    rna_top,
    cmap="YlOrRd",
    linewidths=0.3,
    xticklabels=True,
    yticklabels=True,
    ax=ax,
    cbar_kws={"label": "Relative transcript abundance (%)"}
)
ax.set_title("Top 40 actively transcribed vOTUs", fontsize=13, pad=12)
ax.set_xlabel("Sample", fontsize=10)
ax.set_ylabel("vOTU", fontsize=10)
plt.tight_layout()
plt.savefig("figures/top40_active_vOTUs_heatmap.pdf", dpi=150)
plt.savefig("figures/top40_active_vOTUs_heatmap.png", dpi=300)
print("Heatmap saved")
```

### DNA vs RNA scatter plot (dormant vs active)

```python
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

comp = pd.read_csv("results/dna_rna_comparison.tsv", sep="\t")

colour_map = {
    "Active":                "#2ca02c",
    "Dormant_or_Relic":      "#1f77b4",
    "High_activity_low_copy": "#d62728",
    "Absent":                 "#aaaaaa"
}

fig, ax = plt.subplots(figsize=(7, 6))
for cls, colour in colour_map.items():
    sub = comp[comp["activity_class"] == cls]
    ax.scatter(
        np.log10(sub["mean_DNA_abundance"] + 1e-6),
        np.log10(sub["mean_RNA_abundance"] + 1e-6),
        c=colour, label=cls.replace("_", " "),
        alpha=0.6, s=20, edgecolors="none"
    )

ax.set_xlabel("log₁₀(Mean DNA relative abundance)", fontsize=11)
ax.set_ylabel("log₁₀(Mean RNA relative abundance)", fontsize=11)
ax.set_title("Viral DNA presence vs RNA activity", fontsize=12)
ax.legend(fontsize=9, framealpha=0.8)
plt.tight_layout()
plt.savefig("figures/dna_vs_rna_scatter.pdf", dpi=150)
print("DNA vs RNA scatter saved")
```

### AMG expression bar chart by metabolic category

```python
import pandas as pd
import matplotlib.pyplot as plt

amg_expr = pd.read_csv("results/amg_expression.tsv", sep="\t")
tpm_cols = [c for c in amg_expr.columns if amg_expr[c].dtype == "float64"
            and c not in {"length"}]

if "metabolic_category" in amg_expr.columns and tpm_cols:
    amg_expr["mean_TPM"] = amg_expr[tpm_cols].mean(axis=1)
    cat_summary = (
        amg_expr[amg_expr["mean_TPM"] > 0]
        .groupby("metabolic_category")["mean_TPM"]
        .sum()
        .sort_values(ascending=True)
    )

    fig, ax = plt.subplots(figsize=(8, 6))
    cat_summary.plot(kind="barh", ax=ax, color="#2ca02c", edgecolor="white")
    ax.set_xlabel("Total TPM (all samples)", fontsize=11)
    ax.set_title("AMG expression by metabolic category", fontsize=12)
    plt.tight_layout()
    plt.savefig("figures/amg_expression_by_category.pdf", dpi=150)
    print("AMG category bar chart saved")
```

---

## Part 11 — Caveats and limitations

Scientific maturity means being honest about what your data cannot tell you. These caveats are important to include when reporting results.

### RNA stability and sampling artefacts

RNA degrades rapidly in environmental samples — within minutes to hours at room temperature. The time between collection and preservation (by freezing in liquid nitrogen or adding RNAlater) directly affects which transcripts you capture. Transcripts from rapidly degrading organisms (e.g. after lysis) may be under-represented. Always note your preservation method and time-to-preservation in your methods section.

### Host transcript contamination

In bulk metatranscriptomes, the vast majority of RNA is from bacterial and archaeal hosts, not viruses. Even after rRNA removal, most mRNA reads come from cellular organisms. Viral transcripts are a small and variable fraction. This means your viral TPM values should be interpreted relative to each other and to DNA-based abundance, not in absolute terms.

### Prophage transcription vs lytic transcription

Not all viral RNA signals indicate lytic infection. Integrated prophages are transcribed during lysogeny — particularly their repressor and lysogeny maintenance genes. A temperate phage with high RNA signal may be maintaining lysogeny across a large fraction of the host population rather than actively lysing cells. This is why combining RNA data with lifestyle predictions (Day 4) is essential for correct interpretation.

### Mapping ambiguity

Highly conserved viral genes (terminases, capsid proteins, RNA polymerases) can attract reads from multiple related viruses. If two vOTUs share a conserved gene, reads from that gene region are ambiguous — both genomes get some credit. Use the covered-fraction filter to identify vOTUs where only one or a few genes have coverage (likely false positives) vs. vOTUs with broad genome coverage (likely genuine expression).

### Low viral RNA abundance and detection limits

In some environments and some samples, viral RNA is simply below the detection limit of Illumina sequencing at typical depths (10–50 million reads). A vOTU that appears absent from the RNA data may be genuinely dormant — or it may be active but too rare to sample. Deeper sequencing, viral enrichment, or targeted amplification approaches can help, but are beyond the scope of this series.

---

## Part 12 — Tools summary

| Purpose                     | Tool                    | Notes                                      |
| --------------------------- | ----------------------- | ------------------------------------------ |
| Quality trimming            | fastp                   | fast, good adapter detection               |
| rRNA removal                | SortMeRNA               | in silico rRNA depletion                   |
| Read mapping                | Bowtie2                 | short reads, well-validated for viromics   |
| Read mapping (long reads)   | minimap2                | Nanopore / PacBio RNA reads                |
| Genome-level quantification | CoverM                  | RPKM, relative abundance, covered fraction |
| Gene-level counting         | featureCounts (Subread) | raw counts per gene                        |
| TPM normalisation           | custom Python           | see script in repository                   |
| Viral annotation            | DRAM-v                  | AMG identification (Day 3)                 |
| AMG interpretation          | KEGG / CAZy             | metabolic pathway context                  |
| Visualisation               | matplotlib / seaborn    | heatmaps, scatter plots                    |
| Visualisation               | ggplot2 (R)             | publication-quality figures                |
| Visualisation               | pheatmap (R)            | clustered heatmaps                         |

---

## Part 13 — What you have at the end of Day 6

| File                                        | Contents                                        |
| ------------------------------------------- | ----------------------------------------------- |
| `viral_reference_db.fa`                     | Combined vOTUs + proviruses reference           |
| `*_RNA_sorted.bam`                          | RNA read mappings per sample                    |
| `coverm_RNA_filtered.tsv`                   | RNA-based abundance (RPKM + relative abundance) |
| `featurecounts_RNA_raw.tsv`                 | Raw gene-level read counts                      |
| `gene_expression_TPM.tsv`                   | TPM-normalised gene expression                  |
| `results/active_virus_calls.tsv`            | Per-vOTU activity calls per sample              |
| `results/dna_rna_comparison.tsv`            | DNA vs RNA comparison with activity class       |
| `results/amg_expression.tsv`                | AMG annotations joined with TPM values          |
| `results/top_expressed_amgs.tsv`            | Top 20 expressed AMG functions                  |
| `results/master_ecological_table_final.tsv` | Complete per-vOTU table with RNA activity       |
| `figures/`                                  | Publication-ready heatmaps and scatter plots    |

---

## Closing thoughts: what this series has built

Six posts ago, you had a raw FASTA file of assembled metagenomic contigs. You now have:

- A quality-filtered, clustered set of viral operational taxonomic units
- Taxonomy assigned from two independent methods
- Abundance profiles across all your samples
- A catalogue of auxiliary metabolic genes with confidence flags
- Host predictions from three independent lines of evidence
- Lifestyle predictions separating lytic from lysogenic viruses
- A protein-sharing network placing your viruses in the context of the known viral universe
- And finally — RNA-based evidence distinguishing which of those viruses are genuinely active from which are dormant or relic

This is the complete viromics workflow. The biology it reveals — active viruses reshaping microbial metabolism in real time, seasonal blooms of specific phage lineages, prophages silently waiting for induction — is some of the most dynamic and ecologically meaningful information available from any omics approach.

DNA tells you who is there. RNA tells you who is active. Together, they tell you how the ecosystem actually works.

---

> **Companion repository:** [metagenome-to-viromics](https://github.com/jojyjohn28/metagenome-to-viromics)
> All scripts, installation guides, and toy data for the complete six-day series. Day 6 materials are in `day6/`..

_Today’s image is a simplified toy visualization created for demonstration purposes only._

![see_your_plot](/assets/img/vir-day6.png)
