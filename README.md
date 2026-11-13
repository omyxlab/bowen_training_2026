# Linux for Bioinformatics for Life Scientists

**Bowen University · 16–18 November 2026**

**Theme:** Processing High-Throughput Sequence Data Using Practical Linux Workflows for Agricultural, Health and Biological Sciences.

This repository holds the lessons and commands for a three-day, hands-on training in using the Linux command line for sequence data analysis. It is written for life scientists, including those who have never used a terminal.

We follow **one real dataset**, reads from a 2014 Ebola virus patient sample, from raw data to a finished genome, and then apply the same ideas to a plant virus in cassava sequenced with long reads.

---

## Training program

The schedule for the three days, with the sessions and trainers, is in **[Program.md](Program.md)**.

---

## Start here

1. Complete **[Module 0: Getting Set Up](modules/module0/README.md)** before the training. Follow [Getting_Started.md](modules/module0/Getting_Started.md).
2. Work through the modules in order. Each module's README gives an overview, and each lesson is a single file.

## Modules and lessons

### Module 0: Getting Set Up ([README](modules/module0/README.md))

- [Getting Started](modules/module0/Getting_Started.md)

### Module 1: Welcome and Introduction to Bioinformatics ([README](modules/module1/README.md))

- [1.1 Welcome and course overview](modules/module1/Lesson_1_Welcome_and_Course_Overview.md)
- [1.2 What is bioinformatics?](modules/module1/Lesson_2_What_is_Bioinformatics.md)
- [1.3 Sequencing technologies](modules/module1/Lesson_3_Sequencing_Technologies.md)
- [1.4 Biological file formats](modules/module1/Lesson_4_Biological_File_Formats.md)
- [1.5 Biological databases](modules/module1/Lesson_5_Biological_Databases.md)

### Module 2: Working in the Linux Terminal ([README](modules/module2/README.md))

- [2.1 Meet the terminal](modules/module2/Lesson_1_Meet_the_Terminal.md)
- [2.2 Preparing your environment with Conda](modules/module2/Lesson_2_Preparing_Your_Environment_with_Conda.md)
- [2.3 Files and folders](modules/module2/Lesson_3_Files_and_Folders.md)
- [2.4 Looking inside files](modules/module2/Lesson_4_Looking_Inside_Files.md)

### Module 3: Short-Read Analysis I, from Raw Reads to Alignment ([README](modules/module3/README.md))

- [3.1 Case and project setup](modules/module3/Lesson_1_Case_and_Project_Setup.md)
- [3.2 Quality control and trimming](modules/module3/Lesson_2_Quality_Control_and_Trimming.md)
- [3.3 Mapping reads to a reference](modules/module3/Lesson_3_Mapping_Reads_to_a_Reference.md)
- [3.4 Alignment and coverage](modules/module3/Lesson_4_Alignment_and_Coverage.md)

### Module 4: Short-Read Analysis II, from Alignment to Genome ([README](modules/module4/README.md))

- [4.1 Variant calling](modules/module4/Lesson_1_Variant_Calling.md)
- [4.2 Consensus genome](modules/module4/Lesson_2_Consensus_Genome.md)
- [4.3 Interpretation: genes and identity](modules/module4/Lesson_3_Interpretation_Genes_and_Identity.md)

### Module 5: Long-Read Analysis, the Cassava Virus Example ([README](modules/module5/README.md))

- [5.1 Quality control of Nanopore data](modules/module5/Lesson_1_Quality_Control_of_Nanopore_Data.md)
- [5.2 Host removal by mapping to the cassava genome](modules/module5/Lesson_2_Host_Removal_by_Mapping_to_the_Cassava_Genome.md)
- [5.3 Classification with Kraken2](modules/module5/Lesson_3_Classification_with_Kraken2.md)
- [5.4 Krona plots](modules/module5/Lesson_4_Krona_Plots.md)

### Module 6: Wrap-Up and Next Steps ([README](modules/module6/README.md))

- [6.1 Recap: the short-read pipeline](modules/module6/Lesson_1_Recap_The_Short_Read_Pipeline.md)
- [6.2 Recap: the long-read pipeline](modules/module6/Lesson_2_Recap_The_Long_Read_Pipeline.md)
- [6.3 Capstone brief](modules/module6/Lesson_3_Capstone_Brief.md)
- [6.4 Where next: Bioinformatic Fridays](modules/module6/Lesson_4_Where_Next.md)

### Reference and capstone

- [Command cheat sheet](modules/reference/Command_Cheat_Sheet.md): every command in the course on one page
- [Capstone: a second look at the Ebola sample](capstone/README.md): the take-home project

## Using the JupyterHub (a few participants)

Most participants work on their own laptops (Windows with WSL, Mac or Linux). A few are given a login to a JupyterHub server at **training.omyxlab.com**. If that is you:

- You are given a **username**. The **password you set the first time you log in becomes your password**, so choose one you will remember.
- The `bioinfo` environment is **already built and shared**, and Krona's taxonomy (`ktUpdateTaxonomy.sh`) is **already updated**. You cannot change the environment. Just run `conda activate bioinfo`.
- The `Training` folder and the cassava genome (`GCF_001659605.2_M.esculenta_v8_genomic.fna.gz`) are in your **home folder**. The Kraken2 viral database is at **`/opt/kraken_db/viral`**.
- **Do not** run `conda create`, `conda install`, `ktUpdateTaxonomy.sh`, `sudo`, or any download of the training data, cassava genome or Kraken2 database. Skip Parts 2 to 8 of Module 0.

[Part 11 of the Getting Started guide](modules/module0/Getting_Started.md) has the full list, and each lesson that differs on the hub (2.2, 5.2, 5.3, 5.4) has its own box.

## Training data

The data are published on Zenodo: <https://zenodo.org/records/24057548> (`training.tar.gz`, about 90 MB). Module 0 explains how to download and unpack them. Once unpacked you will have a `Training` folder:

| Folder | Contents |
|---|---|
| `Training/short_reads/paired/` | Illumina paired-end reads from an Ebola virus patient sample (`SRR1553607`) |
| `Training/long_reads/` | Oxford Nanopore reads from cassava (`barcode57`, `barcode58`, `sample3`) |
| `Training/short_reads/unpaired/` | 16S amplicon reads from a gut microbiome study (not used in the main analysis) |

Two larger files are used in Module 5 and are provided separately: the cassava reference genome (about 190 MB) and the Kraken2 viral database (about 590 MB). If you are working on your own computer you can download the cassava genome yourself with `wget -c https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/001/659/605/GCF_001659605.2_M.esculenta_v8/GCF_001659605.2_M.esculenta_v8_genomic.fna.gz`, as shown in Lesson 5.2.

The data themselves are not stored in this repository.

## Tools

The course uses a deliberately small set of commands (`pwd`, `ls`, `cd`, `mkdir`, `cp`, `mv`, `rm`, `head`, `tail`, `less`, `wc`, `grep`, `nano`) together with these bioinformatics programs, installed in a Conda environment called `bioinfo`:

`seqkit`, `fastqc`, `fastp`, `bwa`, `minimap2`, `samtools`, `bcftools`, `NanoPlot`, `entrez-direct`, `kraken2`, `Krona` and `KrakenTools`.

## What this course does not cover

Git and GitHub, shell scripting and remote servers (SSH) are left out on purpose, so that the command line and a complete analysis can be learned properly. The [Bioinformatic Fridays](https://github.com/omyxlab/Bioinformatic_Fridays) series is a good next step.

## Repository layout

```
.
├── README.md
├── Program.md      The schedule for the three days
├── capstone/       The take-home project
└── modules/
    ├── module0/    Getting Set Up
    ├── module1/    Welcome and Introduction to Bioinformatics
    ├── module2/    Working in the Linux Terminal
    ├── module3/    Short-Read Analysis I
    ├── module4/    Short-Read Analysis II
    ├── module5/    Long-Read Analysis
    ├── module6/    Wrap-Up and Next Steps
    └── reference/  Command cheat sheet
```

Every module folder has its own `README.md`, and every lesson is a single Markdown file.

## Getting help

Email **bioinformaticsfridays@gmail.com** with a short description of the problem and a screenshot of any error message.
