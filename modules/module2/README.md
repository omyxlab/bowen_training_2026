# Module 2: Working in the Linux Terminal

**Linux for Bioinformatics for Life Scientists · Bowen University · 16–18 November 2026**

## Overview

This is where you start typing commands. You learn to find your way around the Linux file system, set up the software environment that holds the course tools, organise a project, and look inside large sequencing files safely. Everything here is a skill you will use in every later module.

You work with the real data from the start: by the end of this module you will have copied the Ebola reads into a project folder and counted how many reads they contain.

## Learning objectives

By the end of this module you will be able to:

- Read a terminal prompt and move around with `pwd`, `ls` and `cd`
- Explain absolute and relative paths
- Create, check or repair the `bioinfo` Conda environment that holds the course tools
- Create, copy, move and delete files and folders without losing data
- Use wildcards and the `nano` editor
- Inspect large files with `head`, `tail`, `less`, `wc` and `grep`
- Join commands with a pipe and save results with `>`
- Count the reads in a FASTQ file, including compressed ones

## Lessons

| Lesson | Title | What it covers |
|---|---|---|
| [2.1](Lesson_1_Meet_the_Terminal.md) | Meet the terminal | Prompt, paths, `pwd`, `ls`, `cd`, shortcuts, getting help |
| [2.2](Lesson_2_Preparing_Your_Environment_with_Conda.md) | Preparing your environment with Conda | Environments, channels, creating or verifying `bioinfo` |
| [2.3](Lesson_3_Files_and_Folders.md) | Files and folders | `mkdir`, `cp`, `mv`, `rm`, wildcards, `nano`, `du` |
| [2.4](Lesson_4_Looking_Inside_Files.md) | Looking inside files | `head`, `tail`, `less`, `wc`, `grep`, pipes, `>`, `zcat` |

## Data used in this module

- `Training/short_reads/paired/`: the Ebola virus reads (`SRR1553607_1.fastq` and `_2.fastq`)
- `Training/short_reads/unpaired/`: compressed 16S files, used to practise `zcat`
- `Training/long_reads/`: Nanopore reads, used to compare file sizes and headers

At the end you will have a project folder, `~/ebola_project/raw/`, holding your own copy of the Ebola reads.

## Before you start

Complete [Module 0](../module0/README.md) and read [Module 1](../module1/README.md). You need a working terminal and the `Training` folder.

## What comes next

[Module 3: Short-Read Analysis I](../module3/README.md) checks the quality of the Ebola reads and maps them to a reference genome.
