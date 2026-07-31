---
layout: post
title: "Linux Commands I Use Every Day as a Bioinformatician"
date: 2026-06-08
description: "You do not need to memorize 500 Linux commands to be effective at the command line. You need maybe 20, used well and combined creatively. This post covers the commands I reach for every single day — find, grep, awk, sed, wc, sort, uniq, xargs, parallel, rsync, and a handful of others — with real bioinformatics examples for each one. There is also a short section at the end for Windows users who want to get started without dual-booting: WSL (Windows Subsystem for Linux) puts a full Linux terminal on your Windows machine in about ten minutes."
comments: true
giscus_comments: true
featured: true
categories: [linux]
permalink: /blog/linux-commands-bioinformatician-everyday/
tags: [linux, bash, command-line, bioinformatics, grep, awk, sed, find, parallel, rsync, WSL, Windows, HPC, beginners, productivity]
---

🧬 𝐷𝑎𝑦 90 𝑜𝑓 𝐷𝑎𝑖𝑙𝑦 𝐵𝑖𝑜𝑖𝑛𝑓𝑜𝑟𝑚𝑎𝑡𝑖𝑐𝑠 𝑓𝑟𝑜𝑚 𝐽𝑜𝑗𝑦'𝑠 𝖽𝖾𝗌𝗄

The command line is where bioinformatics actually happens. Genome assemblies, read mapping, taxonomic classification, differential expression — all of it ultimately runs as a series of commands in a terminal. The sooner you get comfortable here, the faster everything else becomes.

This post is not a comprehensive Linux reference. It is a personal list: the commands I open a terminal and immediately use, every day, working with metagenomes, metatranscriptomes, and large genome datasets. Each one comes with real bioinformatics examples, not toy demos. At the end there is a short section for Windows users who want to get started without installing a new operating system — WSL makes this much easier than it used to be.

---

## The mindset: pipes and small tools

Before the commands, the philosophy. Linux is built around the idea that each tool should do one thing well, and tools should be composable — you chain them together with the pipe character `|`, passing the output of one command as input to the next.

```bash
# Instead of a specialized program for "count unique taxa in my file",
# you chain three small tools:
cut -f2 taxonomy.tsv | sort | uniq -c | sort -nr
```

Once this clicks, you stop looking for a specific program for every task and start building exactly what you need from the tools you already have.

---

## 1. `ls` — list directory contents

You already know this one. But a few flags make it genuinely useful:

```bash
ls -lh            # human-readable file sizes
ls -lhS           # sort by size, largest first
ls -lht           # sort by modification time, newest first
ls -1             # one file per line (useful in scripts)
ls *.fastq.gz     # list only matching files
```

In bioinformatics you are constantly checking whether jobs finished, whether output files appeared, and whether they are the right size. `ls -lh` after every pipeline step becomes muscle memory.

---

## 2. `find` — locate files anywhere in the filesystem

`find` is for when you know what you are looking for but not exactly where it is — or when you want to operate on a large set of files that match a pattern.

```bash
# Find all FASTQ files under the current directory
find . -name "*.fastq.gz"

# Find files modified in the last 24 hours (useful after a job finishes)
find . -mtime -1 -name "*.out"

# Find files larger than 1 GB
find . -size +1G

# Find and delete all empty files
find . -empty -type f -delete

# Find all antiSMASH region files across output subdirectories
find antismash_output -name "*region*.gbk"

# Count how many genomes you have
find genomes/ -name "*.fna" | wc -l
```

The `find ... | xargs ...` combination (covered below) is one of the most powerful patterns in bioinformatics shell scripting.

---

## 3. `grep` — search text using patterns

`grep` searches for patterns in files. In bioinformatics this means searching sequence headers, taxonomy strings, log files, result tables — almost everything.

```bash
# Count sequences in a FASTA file
grep -c "^>" assembly.fasta

# Extract all contig headers
grep "^>" assembly.fasta

# Search for a specific genome in a results table
grep "Staphylococcus_capitis" all_results.tsv

# Case-insensitive search
grep -i "candida" taxonomy.txt

# Show lines that DO NOT match (inverse)
grep -v "unclassified" taxonomy.tsv

# Search recursively through all files in a directory
grep -r "ERROR" logs/

# Count fungal reads per phylum in a Kaiju output
grep -c "Eukaryota; Ascomycota" sample.names.out

# Show the line number of matches
grep -n "warning" pipeline.log

# Print only the matching part, not the whole line
grep -o "GC_[0-9]*" gene_clusters.txt | sort | uniq

# Show 2 lines of context before and after a match
grep -B 2 -A 2 "CRITICAL" pipeline.log
```

---

## 4. `awk` — process structured text by column

`awk` is a mini-programming language for working with tabular data. If your file has columns separated by tabs or spaces, `awk` can slice, filter, compute, and reformat it without loading it into R or Python.

```bash
# Print the second column of a TSV file
awk '{print $2}' results.tsv

# Print rows where column 3 is greater than 100
awk '$3 > 100' counts.tsv

# Print rows where column 5 (p-value) is less than 0.05
awk '$5 < 0.05' deseq_results.tsv

# Use tab as delimiter explicitly
awk -F'\t' '{print $1, $4}' metadata.tsv

# Sum the values in column 4
awk '{sum += $4} END {print sum}' coverage.tsv

# Print the genome name and total reads for high-abundance genomes
awk '$3 > 1000 {print $1, $3}' mag_counts.tsv

# Rename sequences in a FASTA: add a prefix to every header
awk '/^>/{print ">PREFIX_" substr($0,2); next} {print}' input.fasta

# Count the number of fields (columns) in a file
awk '{print NF; exit}' metadata.tsv

# Extract genus field from Kaiju taxonomy string (field 6, semicolon-delimited)
awk -F';' '{gsub(/^ +| +$/,"",$6); if($6!="NA" && $6!="") print $6}' sample.names.out
```

The last example is directly from the Kaiju fungal workflow. `awk` is how you parse those semicolon-delimited taxonomy strings without writing a Python script.

---

## 5. `sed` — stream editor for text substitution

`sed` edits text as it streams through — the most common use is find-and-replace, but it can also delete lines, insert lines, and extract ranges.

```bash
# Replace all spaces with underscores in a file
sed 's/ /_/g' names.txt

# Replace only the first occurrence per line (no /g flag)
sed 's/Staphylococcus/S./g' taxonomy.txt

# Delete blank lines
sed '/^$/d' messy_file.txt

# Delete lines containing a pattern
sed '/unclassified/d' taxonomy.tsv

# Print only lines 10 to 20
sed -n '10,20p' large_file.tsv

# Add a header to a file that doesn't have one
sed -i '1i sample\tgroup\tsalinity' metadata.tsv

# Remove Windows line endings (carriage returns) from files transferred from Windows
sed -i 's/\r//' metadata.csv

# In-place editing (modify the file directly, with backup)
sed -i.bak 's/old_name/new_name/g' external-genomes.txt
```

The `\r` removal line is something I run on almost every CSV that came from Excel or was edited on Windows. Invisible carriage return characters cause mysterious failures in tools that are not expecting them.

---

## 6. `wc` — count lines, words, characters

`wc` is simple but essential.

```bash
# Count lines in a file (number of entries in a table)
wc -l results.tsv

# Count sequences in a FASTA (divide by 2 for FASTQ)
grep -c "^>" assembly.fasta

# Count how many samples are in your metadata
wc -l metadata.csv

# Count total characters (bytes) in a file
wc -c large_file.txt

# Count reads in a FASTQ (divide by 4)
wc -l sample_R1.fastq

# Count how many genomes passed quality filtering
wc -l checkm_pass.txt
```

The `-l` flag (count lines) is the one I use ~20 times a day. After any pipeline step that produces a list, I run `wc -l` to confirm the expected number of entries came out.

---

## 7. `sort` — sort lines of a file

```bash
# Sort alphabetically (default)
sort names.txt

# Sort numerically
sort -n counts.txt

# Sort numerically, descending (largest first)
sort -nr counts.txt

# Sort by the second column (tab-delimited)
sort -t$'\t' -k2,2n results.tsv

# Sort by multiple columns: column 1 alphabetically, then column 2 numerically
sort -k1,1 -k2,2n results.tsv

# Remove duplicate lines while sorting
sort -u names.txt

# Sort FASTA by sequence length (useful for assembly QC)
awk '/^>/{if(seq) print length(seq), header; header=$0; seq=""} !/^>/{seq=seq$0} END{print length(seq), header}' \
  assembly.fasta | sort -nr | head -20
```

---

## 8. `uniq` — collapse or count repeated lines

`uniq` only collapses _adjacent_ duplicate lines — which is why it is almost always preceded by `sort`.

```bash
# Count occurrences of each unique value (classic pattern)
sort column.txt | uniq -c

# Show only lines that appear exactly once
sort column.txt | uniq -u

# Show only lines that appear more than once (duplicates)
sort column.txt | uniq -d

# Count unique BGC classes across all genomes
cut -f2 bgc_summary.tsv | sort | uniq -c | sort -nr

# Count how many genomes carry each BGC type
awk -F',' '{print $3}' antismash_results.csv | sort | uniq -c | sort -nr

# Find taxa that appear in multiple samples
cut -f2 taxonomy_all.tsv | sort | uniq -c | sort -nr | head -20
```

---

## 9. `cut` — extract columns from delimited files

`cut` extracts specific columns without loading a full table into memory.

```bash
# Extract the first column of a CSV
cut -d',' -f1 metadata.csv

# Extract columns 1, 3, and 5 from a TSV
cut -f1,3,5 results.tsv

# Extract sample names from a FASTQ file list
ls *.fastq.gz | cut -d'_' -f1

# Get all unique genome names from the first column
cut -f1 gene_clusters.txt | sort | uniq

# Pull run accessions from ENA report
cut -f1 filereport_read_run.tsv | tail -n +2 > run_accessions.txt
```

---

## 10. `head` and `tail` — inspect files without opening them

```bash
# See the first 10 lines (default)
head results.tsv

# See the first 50 lines
head -50 results.tsv

# See the last 20 lines
tail -20 pipeline.log

# Skip the header row (everything from line 2 onward)
tail -n +2 results.tsv

# Watch a log file update in real time (essential during long jobs)
tail -f logs/job_12345.out

# See the header and first few data rows
head -5 results.tsv
```

`tail -f` is how I monitor running SLURM jobs. You open a second terminal, run `tail -f logs/jobname.out`, and watch the log scroll in real time.

---

## 11. `xargs` — pass file lists to commands

`xargs` takes lines from standard input and passes them as arguments to another command. This is how you apply a command to hundreds of files at once without writing a loop.

```bash
# Move all FASTQ files found by find to a new directory
find . -name "*.fastq.gz" | xargs -I{} mv {} /scratch/raw_data/

# Run wc -l on all result files
find results/ -name "*.tsv" | xargs wc -l

# Delete all empty log files
find logs/ -empty -name "*.log" | xargs rm

# Compress all FASTA files
find genomes/ -name "*.fasta" | xargs -n1 gzip

# Run a command on 4 files simultaneously (-P sets parallel processes)
cat sample_list.txt | xargs -n1 -P4 -I{} bash -c 'bwa mem ref.fasta {}_R1.fastq.gz {}_R2.fastq.gz > {}.sam'
```

---

## 12. `parallel` — run jobs in parallel (GNU Parallel)

GNU Parallel is `xargs` with better control over parallelism, output handling, and job management. If `xargs -P` feels limited, `parallel` is the upgrade.

```bash
# Install (if not already available)
conda install -c conda-forge parallel

# Run a command on each sample using 8 cores
parallel -j 8 'kaiju -t nodes.dmp -f db.fmi -i {}_R1.fastq.gz -j {}_R2.fastq.gz -o {}.out' \
  ::: $(cat sample_list.txt)

# Process all FASTA files with 4 cores
parallel -j 4 'prodigal -i {} -o {.}.gff -a {.}.faa' ::: genomes/*.fasta

# Use a two-column input file (sample, output path)
parallel --colsep '\t' -j 8 'antismash {1} --output-dir {2}' \
  :::: sample_antismash_jobs.tsv

# Keep a progress bar visible
parallel --progress -j 8 'gzip {}' ::: *.sam

# Retry failed jobs (useful for jobs with occasional timeouts)
parallel --joblog joblog.txt -j 8 'your_command {}' ::: input_list.txt
```

`parallel` is especially useful on a login node for quick jobs that do not justify a full SLURM submission, or when you have 50 samples to process and want them done in 15 minutes instead of 15 hours.

---

## 13. `rsync` — transfer and sync files reliably

`rsync` is how you move large files between your local machine and an HPC cluster — or between locations on the same cluster — without risking data loss from interrupted transfers.

```bash
# Copy a local directory to the cluster (preserving permissions, showing progress)
rsync -avzP /local/results/ username@hpc.cluster.edu:/scratch/username/results/

# Pull results from the cluster to your local machine
rsync -avzP username@hpc.cluster.edu:/scratch/username/results/ /local/results/

# Dry run: see what would be transferred without actually doing it
rsync -avzn /local/data/ username@hpc:/scratch/data/

# Exclude large intermediate files you don't want to copy
rsync -avzP --exclude "*.bam" --exclude "*.sam" results/ username@hpc:/scratch/results/

# Resume an interrupted transfer (rsync only sends what is missing)
rsync -avzP --partial large_file.tar.gz username@hpc:/scratch/

# Sync a project folder (delete files on destination that are no longer in source)
rsync -avzP --delete project/ username@hpc:/scratch/project/
```

The `-P` flag combines `--progress` (show transfer progress) and `--partial` (keep partial files so an interrupted transfer can resume). I use this combination for every large transfer.

---

## 14. `cat`, `zcat`, `less` — read file contents

```bash
# Print a file to screen
cat file.txt

# Concatenate multiple files
cat sample1.tsv sample2.tsv sample3.tsv > combined.tsv

# Read a gzipped file without decompressing it
zcat sample.fastq.gz | head -8

# Page through a large file interactively (q to quit, / to search)
less results.tsv

# Page through a gzipped file
zless large_results.tsv.gz
```

`less` is how you inspect large files without opening them in a text editor. Once inside: `G` jumps to the end, `g` jumps to the beginning, `/pattern` searches forward, `q` quits.

---

## 15. `chmod`, `screen`, `nohup` — the ones that save you at 11pm

```bash
# Make a script executable
chmod +x run_pipeline.sh
./run_pipeline.sh

# Start a screen session that keeps running after you disconnect
screen -S my_pipeline
# Run your job inside screen
# Detach: Ctrl+A then D
# Reattach later:
screen -r my_pipeline
# List all sessions:
screen -ls

# Run a job that survives logout (alternative to screen)
nohup bash run_pipeline.sh > pipeline.log 2>&1 &
# The & runs it in the background
# nohup prevents it dying when you log out
# pipeline.log captures all output

# Check background jobs
jobs
ps aux | grep run_pipeline
```

`screen` and `nohup` exist for the same reason: SSH connections drop. If your pipeline takes 6 hours and your connection dies after 2, without `screen` or `nohup` the job dies with it. Always wrap long jobs in one of these before logging off.

---

## Combining them: real one-liners from daily work

```bash
# How many reads did each sample produce?
for f in *.fastq.gz; do echo -n "$f: "; zcat "$f" | wc -l | awk '{print $1/4}'; done

# Find samples that produced empty output files (possible failed jobs)
find results/ -name "*.out" -empty | sed 's/results\///' | sed 's/\.out//'

# Get the top 20 most abundant genera across all Kaiju outputs
cat *.names.out | awk -F';' '{gsub(/^ +| +$/,"",$6); if($6!="NA" && $6!="") print $6}' \
  | sort | uniq -c | sort -nr | head -20

# Check how much disk space your project is using
du -sh /scratch/username/project/
du -sh /scratch/username/project/*/ | sort -hr | head -10

# Rename all files matching a pattern
for f in *_R1_001.fastq.gz; do mv "$f" "${f/_R1_001/_R1}"; done

# Extract only classified reads from a Kaiju output
awk '$1 == "C"' sample.out | wc -l
```

---

## For Windows users: getting Linux without leaving Windows

If you are on Windows and want to follow along with bioinformatics tutorials or run tools that only exist on Linux, you have a clean option that does not require dual-booting or a virtual machine: **WSL (Windows Subsystem for Linux)**. It puts a real Linux terminal directly inside Windows, with access to your Windows files.

### What is WSL?

WSL runs a genuine Linux kernel alongside Windows. WSL 2 (the current version) uses a lightweight virtual machine but behaves like a native Linux environment from inside the terminal — you can install `conda`, run `bash` scripts, and use every command in this post exactly as written.

### Installing WSL

Open **PowerShell as Administrator** (right-click → Run as administrator) and run:

```powershell
wsl --install
```

This installs WSL 2 and Ubuntu (the default Linux distribution) in one step. Restart your computer when prompted.

After restarting, Ubuntu will finish setting up and ask you to create a username and password. That username is your Linux identity — it does not need to match your Windows username.

If you want a specific Linux distribution:

```powershell
# See available distributions
wsl --list --online

# Install a specific one
wsl --install -d Ubuntu-22.04
```

### First steps inside WSL

Open the Ubuntu app from the Start menu (or type `wsl` in PowerShell). You are now in a Linux terminal.

```bash
# Update the package list and upgrade installed packages
sudo apt update && sudo apt upgrade -y

# Install common bioinformatics dependencies
sudo apt install -y wget curl git build-essential python3 python3-pip

# Install conda (recommended for managing bioinformatics tools)
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
# Follow the prompts, then restart the terminal

# Verify conda installed
conda --version

# Install a bioinformatics tool (example: FastQC)
conda install -c bioconda fastqc
fastqc --version
```

### Accessing your Windows files from WSL

Your Windows drives are mounted under `/mnt/`:

```bash
# Access your Windows Desktop
ls /mnt/c/Users/YourWindowsUsername/Desktop/

# Copy a file from Windows to your Linux home directory
cp /mnt/c/Users/YourWindowsUsername/Downloads/data.csv ~/data.csv

# Set Windows Desktop as a shortcut (add to ~/.bashrc)
echo 'alias desktop="cd /mnt/c/Users/YourWindowsUsername/Desktop"' >> ~/.bashrc
source ~/.bashrc
desktop    # now takes you straight there
```

### VS Code integration (recommended)

If you use VS Code, install the **WSL extension** (search "WSL" in the Extensions panel). Then from inside WSL:

```bash
code .    # opens VS Code connected to your Linux environment
```

You can edit files, run terminals, and use the integrated debugger — all running natively in Linux, displayed in your Windows VS Code window.

### Common WSL tips

```bash
# Find out which WSL version you are running
wsl --list --verbose     # (from PowerShell)

# Shut down WSL (frees memory)
wsl --shutdown           # (from PowerShell)

# Open Windows Explorer in the current WSL folder
explorer.exe .

# Fix slow file performance: store project files inside WSL (~/) not on /mnt/c/
# Working on /mnt/c/ is slow because it crosses the WSL/Windows boundary
# Store your data at ~/projects/ for full Linux-native speed
```

### WSL limitations to know

- **No GUI tools** by default — bioinformatics command-line tools work perfectly; graphical tools like Geneious or IGV should run on the Windows side instead (WSLg adds limited GUI support on newer Windows 11 builds)
- **File I/O is fastest inside WSL** — if you store large FASTQ files on `/mnt/c/`, operations will be noticeably slower than storing them in `~/` (the Linux filesystem)
- **Not suitable for full HPC workloads** — WSL is for development, testing, and learning. For production runs on large datasets, use a proper Linux cluster

---

## Quick reference card

| Command                | What it does                   | Daily use case                |
| ---------------------- | ------------------------------ | ----------------------------- |
| `ls -lh`               | List files with sizes          | Check job outputs             |
| `find . -name "*.gbk"` | Find files by name             | Locate scattered output files |
| `grep -c "^>"`         | Count FASTA sequences          | Quick QC                      |
| `awk '$5 < 0.05'`      | Filter by column value         | Extract significant results   |
| `sed 's/\r//'`         | Remove Windows line endings    | Fix Excel CSVs                |
| `wc -l`                | Count lines                    | Verify record counts          |
| `sort \| uniq -c`      | Count occurrences              | Summarize taxa or categories  |
| `cut -f1,3`            | Extract columns                | Reshape tables                |
| `head -5 / tail -f`    | Inspect files / watch logs     | QC and monitoring             |
| `xargs -n1 gzip`       | Apply command to file list     | Batch compress files          |
| `parallel -j 8`        | Run jobs in parallel           | Process many samples          |
| `rsync -avzP`          | Transfer files reliably        | HPC ↔ local sync              |
| `screen -S name`       | Persistent terminal session    | Survive SSH disconnection     |
| `nohup ... &`          | Run job detached from terminal | Long jobs on login nodes      |

---

_All commands tested on Ubuntu 22.04 / bash 5.1. WSL installation instructions current as of June 2026 — check `wsl --help` for the latest options on your Windows version._

![see_your_plot](/assets/img/linux_codes.png)
