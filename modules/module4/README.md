# Module 4: Short-Read Analysis II, from Alignment to Genome

**Linux for Bioinformatics for Life Scientists · Bowen University · 16–18 November 2026**

## Overview

You have a checked alignment of the Ebola reads against the reference genome. In this module you use it to answer the questions we started with: **how does this virus differ from the reference, what is its own genome sequence, and what is it?**

You find the variants, build the sample's consensus genome, count the changes gene by gene, and compare the result with the public databases.

## Learning objectives

By the end of this module you will be able to:

- Explain what a variant is, and call variants with `bcftools`
- Read a VCF file with `grep` and `bcftools`, and filter out weak variants
- Build a consensus genome with `samtools consensus` and explain what the `N` characters mean
- Count variants in each gene using the annotation
- Look up the genes in UniProt and identify the virus by comparing the consensus with NCBI BLAST
- Explain how the choice of reference affects the number of variants

## Lessons

| Lesson | Title | What it covers |
|---|---|---|
| [4.1](Lesson_1_Variant_Calling.md) | Variant calling | `bcftools mpileup`, `bcftools call`, reading and filtering a VCF |
| [4.2](Lesson_2_Consensus_Genome.md) | Consensus genome | `samtools consensus`, `N` characters, minimum depth |
| [4.3](Lesson_3_Interpretation_Genes_and_Identity.md) | Interpretation: genes and identity | Variants per gene, UniProt, BLAST |

## Data used in this module

- `~/ebola_project/mapping/SRR1553607.sorted.bam` (and its `.bai`), from Module 3
- `~/ebola_project/reference/NC_002549.1.fasta` and `NC_002549.1.gff3`, from Lesson 3.1

## Tools used

`bcftools`, `samtools`, `seqkit`, and a web browser for UniProt and NCBI BLAST. Run `conda activate bioinfo` first.

## Before you start

Complete [Module 3](../module3/README.md). You need the sorted, indexed BAM and the reference files.

## What comes next

[Module 5: Long-Read Analysis, the Cassava Virus Example](../module5/README.md) applies the same ideas to a different technology and a different virus.
