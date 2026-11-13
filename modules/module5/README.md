# Module 5: Long-Read Analysis, the Cassava Virus Example

**Linux for Bioinformatics for Life Scientists · Bowen University · 16–18 November 2026**

## Overview

You have followed one dataset from raw reads to a finished genome. Now you apply the same ideas to a different problem: **what is in a sample from a cassava plant?** This time the data are Oxford Nanopore **long reads**, the sample is mostly plant, and the question is not "how does this one virus differ from a reference?" but "what viruses are present at all?"

You check the quality of the long reads, remove the reads that come from the plant itself (host removal), classify what is left with Kraken2, and draw an interactive chart with Krona.

## Learning objectives

By the end of this module you will be able to:

- Summarise and judge the quality of Nanopore reads, and say how they differ from Illumina reads
- Explain why host reads are removed, and do it by mapping to the host genome with `minimap2`
- Interpret `samtools flagstat` for long reads, where one read can have several alignments
- Classify reads against a database with `kraken2`, and read a Kraken2 report
- Explain why most reads are "unclassified" and why that is expected
- Turn a Kraken2 report into an interactive Krona chart and interpret it

## Lessons

| Lesson | Title | What it covers |
|---|---|---|
| [5.1](Lesson_1_Quality_Control_of_Nanopore_Data.md) | Quality control of Nanopore data | `seqkit stats`, `NanoPlot`, comparing with Illumina |
| [5.2](Lesson_2_Host_Removal_by_Mapping_to_the_Cassava_Genome.md) | Host removal by mapping to the cassava genome | `minimap2`, `samtools flagstat`, `view -f 4`, `fastq` |
| [5.3](Lesson_3_Classification_with_Kraken2.md) | Classification with Kraken2 | `kraken2`, reading the report, `grep -w` |
| [5.4](Lesson_4_Krona_Plots.md) | Krona plots | `kreport2krona.py`, `ktImportText`, reading the chart |

## Data used in this module

- `Training/long_reads/barcode57.fastq` and `barcode58.fastq`: Nanopore reads from cassava samples
- `Training/long_reads/sample3.fastq`: a third Nanopore sample
- **Two large files supplied by your trainers**: the cassava reference genome (about 190 MB) and the Kraken2 viral database (about 590 MB). Lesson 5.2 and 5.3 explain where to get them. **On the JupyterHub (training.omyxlab.com) both are already there:** the cassava genome is in your home folder as `GCF_001659605.2_M.esculenta_v8_genomic.fna.gz`, and the Kraken2 viral database is at `/opt/kraken_db/viral`. Do not download them. On the hub, Krona's taxonomy is also already updated, so skip that step in Lesson 5.4.

## Tools used

`seqkit`, `NanoPlot`, `minimap2`, `samtools`, `kraken2`, `kreport2krona.py` and `ktImportText`. Run `conda activate bioinfo` first.

## A note on your computer

Mapping to the cassava genome needs **about 5 GB of free memory (RAM)**. On Windows, WSL normally gets only half of your computer's memory, so a laptop with 8 GB may not be enough. Lesson 5.2 explains how to check, and what to do if it fails. If your computer cannot manage it, use the JupyterHub for that step.

## Before you start

Complete [Modules 2](../module2/README.md) to [4](../module4/README.md). You need the `bioinfo` environment.

## What comes next

[Module 6: Wrap-Up and Next Steps](../module6/README.md)
