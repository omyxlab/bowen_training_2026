# Lesson 1.2: What is Bioinformatics?

**Module 1: Welcome and Introduction to Bioinformatics**

## What you will learn

By the end of this lesson you will be able to:

- Define bioinformatics and say where it is used
- Describe the path from a biological sample to an analysed result
- Explain why sequencing data cannot be handled by clicking through files
- Explain what Linux and the command line are, and why bioinformaticians use them
- Say why quality control comes first in every analysis

## What is bioinformatics?

**Bioinformatics** is the use of computers and statistics to store, organise and analyse biological data. Most of the time, that data is **sequence data**: the order of the building blocks (bases) in DNA or RNA.

It sits between biology and computing. A bioinformatician does not need to be a programmer, but needs to be comfortable asking a biological question and then working out which tools and data answer it.

## Why does it matter?

Modern **high-throughput sequencing** machines read millions or even billions of short pieces of DNA or RNA in a single run. Even a small dataset is far too large to read by eye.

Our Ebola example is a **small** one, and it already holds:

- **406,890 reads** (203,445 pairs of reads)
- About **41 million bases** of sequence
- Two files of about 54 MB each

The virus genome those reads came from is only **18,959 bases** long. For comparison, the human genome is around 3 billion bases. Larger genomes and larger projects produce files measured in gigabytes or terabytes.

You cannot open files like these in a word processor or a spreadsheet and learn anything from them. You need tools designed for the job.

### Where is bioinformatics used?

| Field | Example |
|---|---|
| **Health** | Tracking how a virus changes during an outbreak; finding which microbes cause an infection |
| **Agriculture** | Detecting and identifying plant viruses in crops such as cassava; screening crop varieties |
| **Biological sciences** | Studying the microbes in a gut, soil or water sample; comparing genes across species |

These match the three areas in our course theme, and we use an example from health and one from agriculture.

## From sample to result: the big picture

Every sequencing project follows the same general path. Part of it happens at the laboratory bench and part on the computer.

```
 AT THE BENCH
   Sample  ->  Extract DNA or RNA  ->  Prepare a library  ->  Sequence
                                                                  |
                                                                  v
 ON THE COMPUTER
   Raw reads (FASTQ)
        |
        v
   1. Quality control        Are the data good enough to use?
        |
        v
   2. Trimming / cleaning    Remove adapters and low-quality bases
        |
        v
   3. Mapping or assembly    Where do the reads belong?
        |
        v
   4. Analysis               Variants, taxonomy, abundance, ...
        |
        v
   5. Interpretation         What does it mean biologically?
```

Our Ebola virus has an **RNA** genome. Before sequencing, the RNA is converted into DNA (called cDNA), which is why the data are written with the letters A, C, G and T.

The steps on the computer side are called a **pipeline**, because the output of each step becomes the input of the next. This course teaches you to build and run exactly such a pipeline.

## Why the command line?

Almost every bioinformatics tool is run from the **command line**: you type the name of the program and tell it which files to use. There are good reasons for this.

- **The tools are built that way.** Most widely used software (for example the programs for mapping, variant calling and classification) has no point-and-click version.
- **It handles large files.** A command can process a file with millions of reads in seconds, without opening it.
- **It is repeatable.** The commands you type are a written record of what you did. You, or someone else, can run them again and get the same result.
- **It works everywhere.** Powerful remote computers and clusters are almost always used through a command line.

### Linux, the terminal and the shell

These three words are often used together, so here is how they fit:

- **Linux** is an operating system, like Windows or macOS. Most scientific servers and computing clusters run Linux.
- The **terminal** is the window where you type commands.
- The **shell** is the program inside the terminal that reads what you type and runs it. (We use one called *Bash*.)

On Windows, we use **WSL** (Windows Subsystem for Linux) to run a real Linux system inside your computer. That is what you installed in Module 0.

## Look at the data before you trust it

Every analysis starts with one rule: **garbage in, garbage out**. If the raw data are poor, then everything built on top of them is unreliable, however sophisticated the later steps are.

For that reason, the first thing a bioinformatician does with new data is **quality control**. We check how many reads there are, how long they are, and how good each base is. Only then do we move on. You will do this in Module 3.

## The data in this course

| Dataset | What it is | Why we use it |
|---|---|---|
| **Ebola virus** (`SRR1553607`) | Reads from a patient sample collected in **Sierra Leone on 9 June 2014**, sequenced on an Illumina HiSeq 2500 by the Broad Institute | A small viral genome that lets us run a complete analysis quickly |
| **Cassava** (`barcode57`, `barcode58`) | Oxford Nanopore reads from cassava samples | Shows how the same ideas work with long reads and a plant host |

You will meet the details of both in the next lessons.

## Key terms

| Term | Meaning |
|---|---|
| **Bioinformatics** | Using computers and statistics to analyse biological data |
| **High-throughput sequencing** | Reading millions of DNA or RNA fragments at the same time |
| **Read** | One piece of sequence from a sequencing machine |
| **Genome** | The complete genetic material of an organism or virus |
| **Reference genome** | A known sequence we compare our reads against |
| **Pipeline** | A series of analysis steps linked together |
| **Linux** | An operating system widely used for scientific computing |
| **Terminal** | The window in which you type commands |
| **Shell (Bash)** | The program that reads and runs your commands |
| **Quality control (QC)** | Checking that data are good enough to analyse |

## Check your understanding

1. What kind of data does bioinformatics most often deal with?
2. Why is it not practical to analyse a sequencing file by opening it in a spreadsheet?
3. Name two reasons bioinformaticians use the command line.
4. Why do we run quality control before any other analysis?
5. What is the difference between the terminal and the shell?

### Answers

1. Sequence data: the order of bases in DNA or RNA.
2. The files are too large, and spreadsheets and word processors are not designed to analyse sequence data.
3. For example: the tools are built for it, it handles very large files, it is repeatable, and it is used on remote computers.
4. Poor data lead to unreliable results later, so we check quality first.
5. The terminal is the window; the shell is the program inside it that interprets and runs your commands.

## What comes next

[Lesson 1.3: Sequencing technologies](Lesson_3_Sequencing_Technologies.md)
