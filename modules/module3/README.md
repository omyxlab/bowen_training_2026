# Module 3: Short-Read Analysis I, from Raw Reads to Alignment

**Linux for Bioinformatics for Life Scientists · Bowen University · 16–18 November 2026**

## Overview

This module takes the Ebola virus reads from raw files to an alignment against a reference genome. You set up a proper project, check the quality of the data, clean it, map it, and judge how well the genome is covered. These are the first steps of almost every sequencing analysis.

Everything is done on the paired-end Illumina reads from the 2014 Sierra Leone patient sample (`SRR1553607`). The steps run in seconds on a laptop, so you can repeat and explore.

## Learning objectives

By the end of this module you will be able to:

- Organise an analysis project and download a reference genome and its annotation
- Summarise a sequencing run and read a FastQC report
- Trim adapters and low-quality reads with `fastp`, and weigh up what trimming costs
- Map paired reads to a reference with `bwa`, then sort and index the result with `samtools`
- Read a `flagstat` summary and explain why most reads in this run do not map to Ebola virus
- Measure coverage and depth, and look at the reads directly with `samtools tview`

## Lessons

| Lesson | Title | What it covers |
|---|---|---|
| [3.1](Lesson_1_Case_and_Project_Setup.md) | Case and project setup | Project folders, `efetch`, the reference FASTA and GFF3 |
| [3.2](Lesson_2_Quality_Control_and_Trimming.md) | Quality control and trimming | `seqkit stats`, `fastqc`, `fastp` |
| [3.3](Lesson_3_Mapping_Reads_to_a_Reference.md) | Mapping reads to a reference | `bwa index`, `bwa mem`, `samtools sort`, `index`, `flagstat` |
| [3.4](Lesson_4_Alignment_and_Coverage.md) | Alignment and coverage | `samtools coverage`, `depth`, `idxstats`, `tview` |

## Data used in this module

- `~/ebola_project/raw/SRR1553607_1.fastq` and `SRR1553607_2.fastq` (copied in Lesson 2.3)
- The Ebola virus reference genome `NC_002549.1` (FASTA and GFF3), downloaded in Lesson 3.1

## Tools used

`efetch`, `seqkit`, `fastqc`, `fastp`, `bwa`, `samtools`. All are in the `bioinfo` environment, so run `conda activate bioinfo` first.

## Before you start

Complete [Module 2](../module2/README.md). You need the `bioinfo` environment and `~/ebola_project/raw/` holding the two read files.

## What comes next

[Module 4: Short-Read Analysis II](../module4/README.md) finds the differences between the sample and the reference and builds the sample's own genome.
