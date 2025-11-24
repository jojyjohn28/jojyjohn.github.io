---
layout: post
title: "Cleaning and Preparing Genomes for NCBI Submission — A Complete Workflow"
date: 2025-11-21
description: "Step-by-step guide for fixing adapter contamination, removing unwanted contigs, renaming FASTA headers, renumbering contigs, and preparing high-quality genomes for NCBI submission."
comments: true
giscus_comments: true
featured: true
---

🧬 **Cleaning and Preparing Genomes for NCBI Submission**

Today’s post documents the complete workflow I used to prepare **12 bacterial genomes** for NCBI submission.  
Out of **205 genomes** submitted, **12 failed the NCBI contamination screen**:

- **2 genomes** failed due to _unexpected genome size_
- **10 genomes** failed due to _adapter sequences_

I systematically fixed all issues using **seqkit**, **bedtools**, **awk**, and shell scripting on the **Palmetto HPC cluster**.

---

##### ⚙️ **Step 1 — Generate Adapter and Contamination Reports**

Each genome was screened using NCBI’s contamination check.  
The report includes two sections:

#### **🔹 Trim Section (mask these regions)**

Example:
Sequence name | length | span | source
contig02 6271 6238..6271 adaptor:NGB02000.1

#### **🔹 Exclude Section (remove these contigs completely)**

Example:
contig05 3946 prok:g-proteobacteria

Goal:  
✔ Mask adapter regions  
✔ Remove contaminant contigs

---

##### 🧱 **Step 2 — Create BED Files for Adapter Spans**

I extracted adapter coordinates directly from reports:

```bash
for rep in reports/*.csv; do
  base=$(basename "$rep" _report.csv)
  awk -F',' -v OFS="\t" -v g="$base" '
    NR > 1 && $3 ~ /[0-9]+\.\.[0-9]+/ {
      split($3,a,"..");
      print g"_"$1, a[1]-1, a[2];
    }' "$rep" > beds/${base}_adapters.bed
done
```

These .bed files define adapter segments for masking.

##### 🧬 Step 3 — Mask Adapter Regions with Ns

```bash
for fa in assemblies/*.fa; do
  base=$(basename "$fa" .fa)
  bed="beds/${base}_adapters.bed"
  bedtools maskfasta -fi "$fa" -bed "$bed" \
    -fo cleaned/${base}_masked.fa -mc N
done
```

This replaces adapter sequences with Ns (usually 10–40 bases long).

##### 🚫 Step 4 — Remove Contaminant Contigs

From the Exclude section, I created lists of contigs to delete.

```bash
for fa in cleaned/*_masked.fa; do
  base=$(basename "$fa" _masked.fa)
  excl="exclude_lists/${base}_exclude.txt"
  seqkit grep -v -f "$excl" "$fa" > final_ncbi_ready/${base}_clean.fa
done
```

Result:
✔ All contaminated contigs removed safely.

##### 🧾 Step 5 — Restore NCBI Metadata Headers

NCBI FASTA headers include metadata such as strain and organism.
Example correct header:

> contig01 [organism=Staphylococcus ureilyticus] [strain=ABD2_031]
> I restored them using mapping files:

```bash
seqkit replace -p "^(contig[0-9]+)" -r "{kv}" \
  --kv-file rename_maps_fixed/${base}_map.txt --keep-key \
  -o final_ncbi_ready_renamed/${base}_clean_renamed.fa "$fa"
```

##### 🔢 Step 6 — Renumber Contigs Sequentially

Since filtering removes contigs, renumbering ensures they start from 1 again:

```bash
for fa in final_ncbi_ready_renamed/*_clean_renamed.fa; do
  base=$(basename "$fa" _clean_renamed.fa)
  awk -v base="$base" '
    /^>/ {count++; sub(/^>contig[0-9]+/, ">contig" count, $0); print; next}
    {print}
  ' "$fa" > final_ncbi_ready_renumbered/${base}_renumbered.fa
done
```

##### 🧮 Step 7 — QC for Ns and Percent N

I counted Ns and genome length:

```bash
for fa in final_ncbi_ready_renumbered/*.fa; do
  base=$(basename "$fa")
  totalN=$(grep -v ">" "$fa" | tr -cd 'Nn' | wc -c)
  totalLen=$(grep -v ">" "$fa" | tr -d '\n' | wc -c)
  perc=$(awk -v n=$totalN -v l=$totalLen 'BEGIN{printf "%.3f", (n/l)*100}')
  echo -e "${base}\t${totalN}\t${totalLen}\t${perc}%"
done
```

Example:
AWM4_233_renumbered.fa 5807 5927574 0.098%
✔ All genomes were safely below NCBI’s ≤5% Ns requirement (mine were ≤0.1%).

##### 📊 Step 8 — Gap Summary (Runs of N ≥10)

ABD2_031_renumbered.fa 10 391
ABF4_148_renumbered.fa 2 63
AWM4_233_renumbered.fa 182 5807
**This confirms masking was minimal and acceptable.**

##### 🧭 Final Thoughts

This workflow converts problematic assemblies into clean, polished FASTA files ready for NCBI:

✔ Adapter regions masked
✔ Contaminant contigs removed
✔ Metadata restored
✔ Contigs renumbered
✔ QC verified

✨ Result:
12 high-quality genomes successfully resubmitted to NCBI.

![ncbi_submission](/assets/img/ncbi_submission.png)
