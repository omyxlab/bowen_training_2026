# Lesson 5.1: Quality Control of Nanopore Data

**Module 5: Long-Read Analysis, the Cassava Virus Example**

## What you will need

- The `bioinfo` environment active
- `~/Training/long_reads/barcode57.fastq` and `barcode58.fastq`
- A web browser

## What you will learn

By the end of this lesson you will be able to:

- Set up a project folder for the long-read analysis
- Summarise Nanopore reads with `seqkit stats`
- Run NanoPlot and read its summary
- Explain how Nanopore reads differ from the Illumina reads you used before
- Explain why base quality is judged differently for long reads

## Why this matters

You met the Illumina reads in Module 3: exactly 101 bases each, high quality. Nanopore reads are a different kind of data, and the same checks give very different answers. Judging long reads by Illumina standards would lead you to the wrong conclusion, so we check first how these data behave.

You already know the key difference from Lesson 1.3: **Nanopore reads vary hugely in length** and have a **lower per-base accuracy**, in exchange for being far longer.

## Step 1: Set up the project

Make a new project folder for this analysis, with a place for each kind of file, and copy the two barcode files in. We keep your raw data separate, as before.

```
$ cd ~
$ mkdir -p cassava_project/raw cassava_project/reference cassava_project/qc cassava_project/mapping cassava_project/clean cassava_project/kraken
$ cp Training/long_reads/barcode57.fastq Training/long_reads/barcode58.fastq cassava_project/raw/
$ cd cassava_project
$ ls
clean  kraken  mapping  qc  raw  reference
$ ls -lh raw
total 53M
-rwxr-xr-x 1 yourname yourname 25M Nov 10 23:25 barcode57.fastq
-rwxr-xr-x 1 yourname yourname 28M Nov 10 23:25 barcode58.fastq
```

## Step 2: Summarise with `seqkit stats`

```
$ seqkit stats raw/*.fastq
file                 format  type  num_seqs     sum_len  min_len  avg_len  max_len
raw/barcode57.fastq  FASTQ   DNA      8,567  12,176,650        3  1,421.3   34,441
raw/barcode58.fastq  FASTQ   DNA      9,196  13,567,398       13  1,475.4   43,663
```

Compare this with the Illumina table from Lesson 3.2, where every read was exactly 101 bases:

| | Ebola (Illumina) | barcode57 (Nanopore) | barcode58 (Nanopore) |
|---|---|---|---|
| Reads | 203,445 per file | 8,567 | 9,196 |
| Shortest | 101 | **3** | **13** |
| Average | 101 | 1,421 | 1,475 |
| Longest | 101 | **34,441** | **43,663** |

- The reads range from a handful of bases to **more than 43,000**, a thousand-fold spread. Nanopore reads are as long as the DNA molecules that happened to enter the pore.
- Reads of 3 or 13 bases are almost certainly fragments or noise, and are of no use.
- There are far **fewer reads** than in the Illumina run, but each is much longer. About 12 to 13 million bases per sample in total, compared with 41 million for the Ebola run.

Add `-a` for more detail:

```
$ seqkit stats -a raw/*.fastq
```

The columns worth noting:

| File | Q1 | Median (Q2) | Q3 | N50 | GC (%) |
|---|---|---|---|---|---|
| `barcode57.fastq` | 298 | 643 | 1,598.5 | 3,068 | **49.14** |
| `barcode58.fastq` | 321 | 656.5 | 1,622.5 | 3,236 | **38.82** |

- The **median** read is only about 650 bases, much shorter than the average of about 1,450. A few very long reads pull the average up. When lengths vary this much, the median describes a typical read better than the mean.
- **N50** is a measure of read length where half of all the sequenced bases are in reads at least this long. Here it is about 3,100 to 3,200 bases.
- The **GC content differs sharply** between the two samples: 49% in barcode57 and 39% in barcode58. Two samples with such different GC contents contain different mixtures of organisms, which is the first hint that they are not alike. Keep this in mind.

## Step 3: Run NanoPlot

**NanoPlot** makes plots and summaries designed for long reads.

```
$ NanoPlot --fastq raw/barcode58.fastq --outdir qc/nanoplot_barcode58 --threads 4
$ NanoPlot --fastq raw/barcode57.fastq --outdir qc/nanoplot_barcode57 --threads 4
```

| Part | Meaning |
|---|---|
| `--fastq raw/barcode58.fastq` | The reads to analyse |
| `--outdir qc/nanoplot_barcode58` | The folder for the results (it will be created) |
| `--threads 4` | Use 4 processor cores |

Each takes around 15 seconds and prints little. Look at what was made:

```
$ ls qc/nanoplot_barcode58
LengthvsQualityScatterPlot_dot.html
LengthvsQualityScatterPlot_dot.png
LengthvsQualityScatterPlot_kde.html
LengthvsQualityScatterPlot_kde.png
NanoPlot-report.html
NanoStats.txt
Non_weightedHistogramReadlength.html
Non_weightedHistogramReadlength.png
Non_weightedLogTransformed_HistogramReadlength.html
Non_weightedLogTransformed_HistogramReadlength.png
WeightedHistogramReadlength.html
WeightedHistogramReadlength.png
WeightedLogTransformed_HistogramReadlength.html
WeightedLogTransformed_HistogramReadlength.png
Yield_By_Length.html
Yield_By_Length.png
```

(There is also a `.log` file.) The summary numbers are in `NanoStats.txt`, which you can read with `head`:

```
$ head -n 17 qc/nanoplot_barcode58/NanoStats.txt
General summary:         
Mean read length:               1,475.4
Mean read quality:                 15.8
Median read length:               656.5
Median read quality:               17.2
Number of reads:                9,196.0
Read length N50:                3,236.0
STDEV read length:              2,322.4
Total bases:               13,567,398.0
Number, percentage and megabases of reads above quality cutoffs
>Q10:	9160 (99.6%) 13.6Mb
>Q15:	6515 (70.8%) 9.1Mb
>Q20:	2248 (24.4%) 2.5Mb
>Q25:	533 (5.8%) 0.2Mb
>Q30:	172 (1.9%) 0.1Mb
Top 5 highest mean basecall quality scores and their read lengths
1:	44.8 (74)
```

And for barcode57:

```
$ head -n 14 qc/nanoplot_barcode57/NanoStats.txt
General summary:         
Mean read length:               1,421.3
Mean read quality:                 15.3
Median read length:               643.0
Median read quality:               16.6
Number of reads:                8,567.0
Read length N50:                3,068.0
STDEV read length:              2,179.2
Total bases:               12,176,650.0
Number, percentage and megabases of reads above quality cutoffs
>Q10:	8488 (99.1%) 12.1Mb
>Q15:	5570 (65.0%) 7.5Mb
>Q20:	1671 (19.5%) 1.5Mb
>Q25:	367 (4.3%) 0.1Mb
>Q30:	123 (1.4%) 0.0Mb
```

### Reading the quality numbers

NanoPlot summarises quality **per read**: each read has a *mean* quality, and the table counts how many reads reach each cut-off.

- The **median read quality is about 17**: an average error rate of about 2 bases in 100.
- **Almost every read (99%) is at Q10 or better**, and about two thirds to three quarters are at Q15 or better.
- Only about **one read in four or five reaches Q20**, and only about **1 to 2%** reach Q30.

Compare this with Illumina, where 85.9% of **bases** in read 1 were at Q30 or better (Lesson 3.2). Nanopore reads are not as accurate base by base. That is not a fault; it is a property of the technology, and it is the price paid for long reads. It is also why we used different tools for them.

> **Two Q20 numbers, two meanings.** `seqkit stats -a` reports the percentage of *bases* at Q20 or better (about 77 to 79% here). NanoPlot reports the percentage of *reads* whose *average* quality is at least Q20 (about 20 to 24%). They answer different questions, so do not compare them directly.

## Step 4: Look at the plots

Open `qc/nanoplot_barcode58/NanoPlot-report.html` in your browser. As in Lesson 3.2:

- **Windows with WSL:** run `explorer.exe qc` and open the folder `nanoplot_barcode58`
- **Mac:** `open qc/nanoplot_barcode58`
- **JupyterHub:** find the file in the file browser

The report shows four useful plots:

| Plot | What it shows |
|---|---|
| Histogram of read lengths | Most reads are short (under 2,000 bases), with a long tail of much longer ones |
| Histogram, log scale | The same on a logarithmic axis, which spreads the shorter reads out so you can see the distribution |
| Weighted histogram | Weights each length by the bases it contributes: shows that the **long reads hold much of the data** |
| Length against quality | Whether longer reads are better or worse. Here the longest reads are a little **lower** in quality: the five longest reads in barcode58 have mean qualities of 11 to 14, below the median of 17 |

The **log-scale histogram** is usually the most informative for Nanopore data, because the lengths span such a wide range.

## What we have learned

| Question | Answer |
|---|---|
| How many reads, and how long? | About 8,600 and 9,200 reads; median about 650 bases; a few reads longer than 30,000 |
| How good are they? | Median read quality about 17 (about 98% accuracy), far lower per base than Illumina |
| Are the two samples alike? | No: their GC contents differ (49% against 39%) |

There is nothing wrong with these data. The next step is to deal with the biggest contaminant of all: the plant itself.

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `NanoPlot: command not found` | The environment is not active | `conda activate bioinfo` |
| NanoPlot runs for a long time with no output | It is plotting; it is normally quick | Wait 30 seconds; press Ctrl+C if it takes minutes |
| `Error: ... outdir` | The output folder name is wrong | Check the `--outdir` option |
| NanoPlot options are not recognised | A different NanoPlot version | Run `NanoPlot --help`; look for the output-folder option |
| Different numbers in NanoStats | A different NanoPlot version | The read lengths must match; small quality differences are normal |
| The plots will not open | Browser path problem | Use `explorer.exe qc` and double-click the HTML file |

## Practice check

1. Why is the median read length more useful than the mean for these data?
2. Roughly what share of reads in barcode58 reach Q20 (mean read quality)?
3. Why should you not compare Illumina and Nanopore reads by per-base quality alone?
4. What does the difference in GC content between barcode57 and barcode58 suggest?

### Answers

1. A few very long reads pull the mean up (1,475) while most reads are short. The median (about 650) describes a typical read better.
2. About 24% (2,248 of 9,196 reads).
3. They are different technologies: Nanopore reads are less accurate per base but much longer. Each should be judged against what it is designed to do.
4. That the two samples contain different mixtures of organisms, since organisms differ in GC content.

## Summary of commands

| Command | What it does |
|---|---|
| `seqkit stats files` | Reads, lengths and totals |
| `seqkit stats -a files` | Adds the length quartiles, N50, quality and GC |
| `NanoPlot --fastq file --outdir dir --threads N` | Makes long-read QC plots and statistics |
| `head -n 17 dir/NanoStats.txt` | Reads the NanoPlot summary |

## What comes next

[Lesson 5.2: Host removal by mapping to the cassava genome](Lesson_2_Host_Removal_by_Mapping_to_the_Cassava_Genome.md)
