# Lesson 1.1: Welcome and Course Overview

**Module 1: Welcome and Introduction to Bioinformatics**

Welcome to *Linux for Bioinformatics for Life Scientists*. 

Theme: **processing high-throughput sequence data using practical Linux workflows for agricultural, health and biological sciences**.

## What you will learn

By the end of this lesson you will be able to:

- Describe the story the course follows from start to finish
- Say what you will be able to do by the end of the training
- Explain how the sessions work and where to find the lessons
- Describe the training data you will use

## Who this course is for

This course is for life scientists. It is meant for people who work in microbiology, biochemistry, plant science, health sciences and related fields. **You do not need any programming experience, and you do not need to have used a terminal before.** If you have never seen a command line, you are exactly who we have designed this for.

## The storyline: from raw reads to a genome

Instead of learning commands in isolation, we follow **one real dataset** all the way through an analysis.

The dataset is a set of sequencing reads from a **patient sample taken during the 2014 Ebola outbreak in Sierra Leone**. Our question is a simple one:

> *What virus is in this sample, and how does its genome differ from a known reference?*

Along the way you will do what a bioinformatician does with a new dataset:

1. Learn the terminal and set up your software
2. Look inside the raw sequencing files
3. Check the **quality** of the data and clean it
4. **Map** the reads to a reference genome
5. Check how well the genome is covered
6. Find the **variants** (differences from the reference)
7. Build the sample's own **consensus genome**
8. Work out what the differences mean

Then we use the same ideas on a **different problem**: a plant virus in cassava, sequenced with a different technology (long reads). There we remove the host plant's reads and ask *what else is in the sample?* using a classification tool and an interactive plot.

By the end, you will have run, by yourself, a full analysis from raw reads to final result, using only the command line.

## What this course covers, and what it leaves out

**Covered**

- Moving around the Linux file system and working with files and folders
- Installing bioinformatics software with Conda
- Quality control and trimming of sequencing reads
- Read mapping, coverage, variant calling and consensus sequences
- Host removal and taxonomic classification of long reads

**Deliberately left out**

- Git and GitHub
- Writing shell scripts (loops, variables and so on)
- Remote servers and SSH

These are all useful, and you will be ready for them afterwards. Doing less, properly, is better in three days than trying to cover everything.

## How the course works

- **You type the commands yourself**, alongside the trainer, one step at a time. There is no separate practice time: the lesson is the practice.
- **The slides** explain the ideas behind each step. **The lessons** in this repository give you the exact commands and the output you should expect, so you can repeat everything at home.
- **Mistakes are normal.** Typing errors, missing spaces and "command not found" are what everyone sees at first. When something looks different from the lesson, check the "If something looks different" section in that lesson.
- **Ask questions.** If you are stuck, raise your hand or ask the person next to you.

## The training data

All the data you need is in the `Training` folder you downloaded in [Module 0](../module0/Getting_Started.md).

> **Using the JupyterHub (training.omyxlab.com)?** The `Training` folder is **already in your home folder**, so there is nothing to download or unpack. Your `bioinfo` environment is also ready, and you cannot change it. Please read [Part 11 of the Getting Started guide](../module0/Getting_Started.md) for a short list of commands **not** to run on the server.

| Folder or file | What it is | Technology |
|---|---|---|
| `short_reads/paired/SRR1553607_1.fastq` and `_2.fastq` | Reads from the Ebola virus patient sample (the main dataset) | Illumina, paired-end |
| `long_reads/barcode57.fastq` and `barcode58.fastq` | Reads from cassava samples (the second example) | Oxford Nanopore |
| `long_reads/sample3.fastq` | An additional Nanopore sample | Oxford Nanopore |
| `short_reads/unpaired/*.fastq.gz` | 16S amplicon reads from a gut microbiome study (not used in the main analysis) | Illumina |

## Before you start: setup checklist

Open your terminal (Ubuntu on Windows, or the Terminal on a Mac) and check that:

- [ ] A terminal window opens and shows a prompt
- [ ] Typing `ls ~/Training` shows two folders: `long_reads` and `short_reads`

Do not worry if you have not yet created the `bioinfo` environment specifed in  Module 0. We do that together in Module 2.

## How to get help

- Ask a trainer or the person next to you.
- Email **bioinformaticsfridays@gmail.com** with a description of the problem and a screenshot of any error message.

## Key terms

| Term | Meaning |
|---|---|
| **Read** | One piece of sequence produced by a sequencing machine |
| **Reference genome** | A known, trusted sequence that we compare our data against |
| **Pipeline** | A series of analysis steps, where the output of one is the input of the next |
| **Terminal** | The window where you type commands |
| **Command line** | Controlling a computer by typing commands instead of clicking |

## What comes next

[Lesson 1.2: What is bioinformatics?](Lesson_2_What_is_Bioinformatics.md)
