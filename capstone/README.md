# Capstone: A Second Look at the Ebola Sample

**Linux for Bioinformatics for Life Scientists · Bowen University · 16–18 November 2026**

This is a take-home project. You will repeat the Ebola analysis with a **different reference genome** and find out what changes, and why. It uses only commands you have already met. Work at your own pace over **one to two weeks**.

## The question

In Module 4 you compared the Ebola sample with a reference isolated in **1976** and found hundreds of differences. You explained this by saying the reference was a distant relative of the sample. That was an explanation, but we did not test it.

> **If we use a reference that is closer to the sample, does the number of differences really drop? And what else changes, and what stays the same?**

You will answer this with data, and write up what you find.

## Before you start

You need:

- The `bioinfo` environment (`conda activate bioinfo`)
- Your **trimmed reads** from Lesson 3.2: `~/ebola_project/trimmed/SRR1553607_1.trim.fastq` and `SRR1553607_2.trim.fastq`
- Your results from Modules 3 and 4, to compare against: the `flagstat` and `coverage` output, the VCF files and the consensus

If you no longer have the trimmed reads, run the pipeline for Modules 2 to 4 again from the start. It takes only a few minutes of computing time, and the repetition is useful practice. The [recap in Lesson 6.1](../modules/module6/Lesson_1_Recap_The_Short_Read_Pipeline.md) lists every command in order.

## The reference genomes

You will use two new references, both downloaded with `efetch` as you did in Lesson 3.1:

| Accession | What it is | Why we use it |
|---|---|---|
| `KU143793.1` | *Zaire ebolavirus* isolate Makona-S26, a complete genome from **Sierra Leone, 2014** | A reference close to our sample, in time and place |
| `NC_001467.1` | *African cassava mosaic virus*, DNA-A, a plant virus | An **unrelated** virus: a negative control |

## The tasks

Work in a new folder inside your project, called `capstone`:

```
$ cd ~/ebola_project
$ mkdir -p capstone capstone_results
```

Keep the outputs you are asked for in `capstone_results`, using `>` to save text output to files.

### Part 1: A closer reference

1. **Download** the reference `KU143793.1` as a FASTA file into the `capstone` folder. Check it with `seqkit stats`: how long is it?
2. **Index** it, **map** your trimmed reads to it, **sort** and **index** the BAM file. (Lesson 3.3. Use the one-line `bwa mem ... | samtools sort` form.)
3. **Measure the mapping.** Save the `samtools flagstat` output and the `samtools coverage` output to files in `capstone_results`.
4. **Call the variants** with `bcftools mpileup | bcftools call`, then **filter** them with the same thresholds as in Lesson 4.1 (`QUAL>=30 && INFO/DP>=10`). Save both VCF files.
5. **Build the consensus** with `samtools consensus -a`. Count the `N` characters, and rename the sequence header so that it does not carry the reference's name. Save the FASTA.

### Part 2: A negative control

6. A **control** checks that your method is not producing results by accident. Download the *African cassava mosaic virus* sequence `NC_001467.1`, and map the same trimmed Ebola reads to it, in the same way. **How many reads map?** Save the `flagstat` output.

### Part 3: Compare and explain

7. Fill in the file `findings.txt` (see the template below) with your numbers and your answers to the questions.

### Optional extension

8. For the cassava data, build a single Krona chart of all three samples from Lesson 5.4, open it, and write two sentences describing what you see for each sample.

## Hints

- You have done every one of these steps before. Look back at: Lesson 3.1 (`efetch`), 3.3 (`bwa`, `samtools`), 3.4 (`coverage`), 4.1 (variants), 4.2 (consensus).
- Give every new file a name that says which reference it belongs to, for example `makona.sorted.bam` and `makona.vcf`, so you never confuse them with the Module 3 and 4 files.
- Compare **like with like**: the same reads, the same trimming, the same filter thresholds. Only the reference should change.
- Remember to read `primary mapped`, not `mapped`, in `flagstat`.
- If a command fails, read the error message and check the "If something looks different" section of the matching lesson.

## What to write: `findings.txt`

Create a plain text file with `nano` and answer these questions, in your own words, in complete sentences. Copy the template from [findings_template.txt](findings_template.txt) if you wish.

1. Which reference did you use, and how long is it?
2. How many reads mapped to it (`primary mapped`), and what percentage is that? How does this compare with the 1976 reference?
3. What were the genome coverage and the mean depth? How do they compare with the 1976 reference?
4. How many variants did you call (before and after filtering)? How does this compare with the 559 and 394 you found before?
5. **Explain the difference** between the numbers of variants. What does it tell you about how the choice of reference affects an analysis?
6. How many `N` characters does the new consensus have? Is that more, fewer or the same as before, and why might that be?
7. What happened in the negative control? Why does that matter?
8. Write a short conclusion (about five sentences). What is in the sample, how close is it to the reference you used, and what would you want to do next?

You are being asked for **understanding**, not just numbers. A good answer explains why a number is what it is.

## What to submit

Your submission is **one compressed file** containing:

- `findings.txt`
- The text files you saved (`flagstat` and `coverage` output)
- The two VCF files and the consensus FASTA
- A record of the commands you ran (see below)

**Do not** include the BAM files or the read files: they are large and not needed.

### Save a record of your commands

The terminal remembers the commands you typed. Save them to a file:

```
$ history > capstone_results/commands_history.txt
```

### Pack everything into one file

First check what is in the folder, then compress it. Replace `yourname` with your own name (no spaces):

```
$ ls capstone_results
$ tar -czf yourname_capstone.tar.gz capstone_results
$ ls -lh yourname_capstone.tar.gz
```

`tar -czf` creates (`c`) a compressed (`z`) archive file (`f`) from the folder. The result should be small, usually well under 1 MB. If it is many megabytes, you have included a BAM or read file by mistake.

### Send it

Submission link: **[TO BE ADDED BY THE TRAINERS]**

Deadline: **[TO BE ADDED BY THE TRAINERS]**

If you cannot use WSL any more, or you get stuck, email **bioinformaticsfridays@gmail.com** with a description of the problem and a screenshot of any error message.

## Checklist before you submit

- [ ] I used the trimmed reads, not the raw reads
- [ ] I only changed the reference, not the settings
- [ ] `findings.txt` answers all eight questions with my own numbers
- [ ] The negative control was run and reported
- [ ] The consensus header was renamed
- [ ] The commands history is saved
- [ ] The `.tar.gz` file is small and contains no BAM or FASTQ files

## What you will have shown

If you complete this, you have:

- Run the full short-read workflow independently, from a reference you chose yourself
- Seen directly how much the **choice of reference** changes the results
- Used a negative control to check your method
- Written up an analysis clearly and reproducibly

Good luck. When you are done, a good next step is [Bioinformatic Fridays](https://github.com/omyxlab/Bioinformatic_Fridays).
