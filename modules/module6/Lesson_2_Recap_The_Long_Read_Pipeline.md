# Lesson 6.2: Recap, The Long-Read Pipeline

**Module 6: Wrap-Up and Next Steps**

## What you will need

- Nothing new. This lesson summarises Module 5.
- If you want to run it yourself: the `bioinfo` environment, the `Training` folder, the cassava genome and the Kraken2 viral database.

## What you will learn

By the end of this lesson you will be able to:

- Describe the long-read workflow as a sequence of steps, each with a purpose and a check
- Compare it with the short-read workflow: what is the same, and what changes
- Run the whole workflow again
- Explain why the questions asked of the two datasets were different

## The workflow on one page

The cassava analysis asked a different question from the Ebola one: not "how does this virus differ from a reference?" but "**what viruses are in this sample?**"

```
raw reads (FASTQ, Nanopore)
    |  quality control          seqkit stats, NanoPlot
    v
    |  map to the HOST genome   minimap2 | samtools sort
    |  keep the unmapped reads  samtools view -f 4, samtools fastq
    v
non-host reads (FASTQ)
    |  classify                 kraken2 (viral database)
    v
report (what is in the sample)
    |  draw it                  kreport2krona.py, ktImportText
    v
interactive chart (HTML)
```

| Step | Lesson | What goes in | What comes out | What you check |
|---|---|---|---|---|
| Quality control | 5.1 | Raw FASTQ | Statistics and plots | Median length about 650; median quality about 17; GC 49% against 39% |
| Host removal | 5.2 | FASTQ, cassava genome | Non-host FASTQ | `primary mapped`: 724 (7.87%); 8,472 reads kept |
| Classification | 5.3 | Non-host FASTQ, database | Report | 1.18% classified, mostly begomoviruses |
| Visualisation | 5.4 | Report | Krona HTML chart | Click `Viruses` to zoom in |

## Every command, in order

**Project and files (Lesson 5.1)**

```
conda activate bioinfo
cd ~
mkdir -p cassava_project/raw cassava_project/reference cassava_project/qc cassava_project/mapping cassava_project/clean cassava_project/kraken
cp Training/long_reads/barcode57.fastq Training/long_reads/barcode58.fastq cassava_project/raw/
cd cassava_project
```

Put the cassava genome in `reference/` and the Kraken2 database in `~/databases/kraken2_viral`, as your trainers explained in Lessons 5.2 and 5.3.

> **On the JupyterHub (training.omyxlab.com)?** Copy the genome from your home folder with `cp ~/GCF_001659605.2_M.esculenta_v8_genomic.fna.gz reference/`, and make the shortcut to the database with `mkdir -p ~/databases` and `ln -s /opt/kraken_db/viral ~/databases/kraken2_viral` (skip this if you made it in Lesson 5.3). Do not download either one.

**Quality control (Lesson 5.1)**

```
seqkit stats raw/*.fastq
seqkit stats -a raw/*.fastq
NanoPlot --fastq raw/barcode58.fastq --outdir qc/nanoplot_barcode58 --threads 4
NanoPlot --fastq raw/barcode57.fastq --outdir qc/nanoplot_barcode57 --threads 4
head -n 17 qc/nanoplot_barcode58/NanoStats.txt
```

**Host removal (Lesson 5.2)**

```
free -h
minimap2 -t 4 -ax map-ont reference/GCF_001659605.2_M.esculenta_v8_genomic.fna.gz raw/barcode58.fastq 2>/dev/null | samtools sort -o mapping/barcode58_vs_cassava.sorted.bam -
samtools index mapping/barcode58_vs_cassava.sorted.bam
samtools flagstat mapping/barcode58_vs_cassava.sorted.bam | head -n 8
samtools view -b -f 4 mapping/barcode58_vs_cassava.sorted.bam > mapping/barcode58_unmapped.bam
samtools fastq mapping/barcode58_unmapped.bam > clean/barcode58_nonhost.fastq
seqkit stats raw/barcode58.fastq clean/barcode58_nonhost.fastq
```

Repeat the five mapping and extraction lines for `barcode57`.

**Classification (Lesson 5.3)**

```
kraken2 --db ~/databases/kraken2_viral --threads 4 --report kraken/barcode58.report --output kraken/barcode58.kraken clean/barcode58_nonhost.fastq
kraken2 --db ~/databases/kraken2_viral --threads 4 --report kraken/barcode57.report --output kraken/barcode57.kraken clean/barcode57_nonhost.fastq
kraken2 --db ~/databases/kraken2_viral --threads 4 --report kraken/sample3.report --output kraken/sample3.kraken ~/Training/long_reads/sample3.fastq
head -n 14 kraken/barcode58.report
grep -w "S" kraken/barcode58.report
```

**Charts (Lesson 5.4)**

```
kreport2krona.py -r kraken/barcode58.report -o kraken/barcode58.krona.txt
kreport2krona.py -r kraken/barcode57.report -o kraken/barcode57.krona.txt
kreport2krona.py -r kraken/sample3.report -o kraken/sample3.krona.txt
ktImportText kraken/barcode57.krona.txt kraken/barcode58.krona.txt kraken/sample3.krona.txt -o kraken/all_samples.krona.html
```

## The two workflows side by side

| | Short-read (Ebola) | Long-read (cassava) |
|---|---|---|
| **Technology** | Illumina, paired-end | Oxford Nanopore |
| **Reads** | 101 bases, all the same length | A few bases to 43,000 |
| **Question** | How does this virus differ from a reference? | What is in the sample? |
| **Mapper** | `bwa mem` | `minimap2 -ax map-ont` |
| **Mapped to** | The Ebola genome (to keep those reads) | The cassava genome (to **remove** those reads) |
| **QC tools** | `fastqc`, `fastp` | `NanoPlot` |
| **Trimming** | Yes (adapters, low quality) | Not done here |
| **Main analysis** | Variants and a consensus | Classification |
| **Result** | A genome, and its differences | A list of viruses and a chart |
| **Share of data that was the target** | 1.2% were Ebola | About 1% were classified viruses |

### What is the same

- Both start by **looking at the data** (QC) before anything else.
- Both use **mapping** and read the same `samtools flagstat` output.
- Both use **pipes**, **sorted and indexed BAM files** and the same `samtools` tools.
- In both, **most of the data were not the target**, and the result still came out clearly.
- Both **ask you to be careful** about what the numbers mean: `primary mapped`, rare taxa, and the reference or database you chose.

### What is different, and why

- **Different data, different tools.** Long reads have a different error pattern, so we use `minimap2` and `NanoPlot` rather than `bwa` and `FastQC`.
- **Different questions, different analyses.** If you know what virus you are looking for and have a reference, you map to it and find variants. If you do not know, you classify.
- **Mapping used in opposite directions.** For Ebola we kept the reads that mapped. For cassava we kept the reads that did **not**.

## Check your understanding

1. Why do we map the cassava reads to the cassava genome, when we are looking for viruses?
2. Which command picks out the species rows in a Kraken2 report?
3. Name two things that are the same in the short-read and long-read workflows, and one that differs.
4. Why was `sample3` not put through host removal?

### Answers

1. To find and remove the host reads. What does not map to cassava is, by definition, not cassava, and goes forward to be classified.
2. `grep -w "S" report`.
3. For example: both begin with quality control and both use sorted, indexed BAM files and the same `samtools` commands. They differ in, for example, the mapper (`bwa` against `minimap2`) or the question asked.
4. It was an extra sample, classified directly to see what it contained. It turned out that 87% of its reads were a single virus, so its make-up was clearly very different from the other two.

## What comes next

[Lesson 6.3: Capstone brief](Lesson_3_Capstone_Brief.md)
