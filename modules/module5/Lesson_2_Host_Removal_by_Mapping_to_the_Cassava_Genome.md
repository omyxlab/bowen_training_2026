# Lesson 5.2: Host Removal by Mapping to the Cassava Genome

**Module 5: Long-Read Analysis, the Cassava Virus Example**

## What you will need

- The `bioinfo` environment active
- `~/cassava_project/raw/barcode57.fastq` and `barcode58.fastq` (Lesson 5.1)
- The **cassava reference genome**, from your trainers (see Step 1)
- **About 5 GB of free memory** (see Step 2)

## What you will learn

By the end of this lesson you will be able to:

- Explain why reads from the host are removed before looking for viruses
- Map long reads to a reference genome with `minimap2`
- Explain why a long read can have several alignments, and read `flagstat` correctly for them
- Extract only the reads that did **not** map, as a new FASTQ file
- Measure how many reads were removed

## Why this matters

A sample taken from a cassava plant contains DNA from the plant itself, the **host**, as well as from any viruses or other organisms in it. The question we are asking is about what *else* is there. Reads that came from the cassava genome cannot answer it, and in many samples they are the great majority of the data. They also slow down every later step.

So we take them out. The trick is simple: **map all the reads to the host genome, and keep the ones that do not map.** What is left is, by definition, not cassava.

```
all reads  ->  map to the cassava genome  ->  mapped reads: host (discard)
                                           ->  unmapped reads: not host (keep)
```

## Step 1: Get the cassava reference genome

We use the NCBI RefSeq genome of cassava (*Manihot esculenta*), assembly `GCF_001659605.2`. It is large: about 190 MB compressed and 640 million bases, in 86 sequences (18 chromosomes and smaller pieces). **Do not download it on the course network.** Your trainers will give it to you (on the JupyterHub, or on a USB drive or a shared folder). Copy the file into your project:

```
$ cp /path/from/trainers/GCF_001659605.2_M.esculenta_v8_genomic.fna.gz ~/cassava_project/reference/
$ ls -lh ~/cassava_project/reference
total 180M
-rw-r--r-- 1 yourname yourname 180M Nov 10 23:25 GCF_001659605.2_M.esculenta_v8_genomic.fna.gz
```

(Ask your trainers for the correct path. On the JupyterHub the file is already in place.) If you are working at home on a good connection, you can download it yourself:

```
$ cd ~/cassava_project/reference
$ wget "https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/001/659/605/GCF_001659605.2_M.esculenta_v8_genomic.fna.gz"
$ ls -l
```

The file should be **188,402,359 bytes**.

The `.gz` ending means the file is **compressed**. You do **not** need to unpack it: `minimap2` reads compressed FASTA directly. (Unpacked it would be 619 MB.)

You can see what is inside it without unpacking:

```
$ zcat GCF_001659605.2_M.esculenta_v8_genomic.fna.gz | grep ">" | head -n 3
>NC_035161.2 Manihot esculenta cultivar AM560-2 chromosome 1, M.esculenta_v8, whole genome shotgun sequence
>NC_035162.2 Manihot esculenta cultivar AM560-2 chromosome 2, M.esculenta_v8, whole genome shotgun sequence
>NC_035163.2 Manihot esculenta cultivar AM560-2 chromosome 3, M.esculenta_v8, whole genome shotgun sequence
$ cd ~/cassava_project
```

## Step 2: Do you have enough memory?

Mapping to a genome of this size needs a lot of memory (RAM): **about 4.5 to 5 GB**. When we measured it, the peak was 4.4 GB. Check how much your computer has:

```
$ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.6Gi       640Mi       4.6Gi       3.0Mi       2.4Gi       6.8Gi
```

Look at the **available** column. You need **at least 5 GB** there.

**On Windows with WSL**, the amount available to Linux is capped at **half of your computer's memory** by default. So a laptop with 8 GB of RAM gives Linux only about 4 GB, which is not enough, and the mapping will be stopped partway with a one-word message: `Killed`. If your available memory is under 5 GB:

- **Use the JupyterHub for this lesson.** This is the simplest option.
- **Or raise the limit**, if your laptop has at least 12 GB of RAM. In Windows, create a file called `.wslconfig` in your user folder (`C:\Users\YourName\.wslconfig`) containing these two lines:

  ```
  [wsl2]
  memory=7GB
  ```

  Then, in PowerShell, run `wsl --shutdown`, and reopen Ubuntu. Run `free -h` again to check.

## Step 3: Map the reads to the cassava genome

```
$ minimap2 -t 4 -ax map-ont reference/GCF_001659605.2_M.esculenta_v8_genomic.fna.gz raw/barcode58.fastq 2>/dev/null | samtools sort -o mapping/barcode58_vs_cassava.sorted.bam -
```

This is the same pattern as in Lesson 3.3, with `minimap2` in place of `bwa`.

| Part | Meaning |
|---|---|
| `minimap2` | A mapper designed for long reads |
| `-t 4` | Use 4 processor cores |
| `-ax map-ont` | `-a` means write SAM output. `-x map-ont` selects the settings for **Oxford Nanopore** reads |
| `reference/...fna.gz` | The cassava genome (compressed) |
| `raw/barcode58.fastq` | The reads |
| `2>/dev/null` | Hide the progress messages |
| `\| samtools sort -o ... -` | Pass the output straight into `samtools sort`, which writes a sorted BAM (Lesson 3.3) |

It takes **about 40 seconds**, because it first reads the whole genome. Nothing is printed while it works. Be patient, and when the prompt returns, index the result:

```
$ samtools index mapping/barcode58_vs_cassava.sorted.bam
$ ls -lh mapping
total 12M
-rw-r--r-- 1 yourname yourname  12M Nov 10 23:25 barcode58_vs_cassava.sorted.bam
-rw-r--r-- 1 yourname yourname 305K Nov 10 23:25 barcode58_vs_cassava.sorted.bam.bai
```

## Step 4: How many reads were host?

```
$ samtools flagstat mapping/barcode58_vs_cassava.sorted.bam | head -n 8
15042 + 0 in total (QC-passed reads + QC-failed reads)
9196 + 0 primary
2849 + 0 secondary
2997 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
6570 + 0 mapped (43.68% : N/A)
724 + 0 primary mapped (7.87% : N/A)
```

Look carefully, because this is easy to misread:

| Line | Meaning |
|---|---|
| `9196 primary` | The 9,196 reads we started with |
| `2849 secondary` | Extra alignments: the same read matching **another place** in the genome as well (cassava has many repeated sequences) |
| `2997 supplementary` | Extra alignments for reads that match in **separate pieces** (a long read can land in two or more places) |
| `15042 in total` | Every alignment record: the reads plus all the extras |
| `6570 mapped (43.68%)` | Alignment **records** that mapped. **Misleading** |
| **`724 primary mapped (7.87%)`** | **Reads that mapped to cassava. This is the number to quote** |

If you read the `43.68%` line you would think nearly half the data is host. The truth is **724 reads, 7.9%**. One long read can generate several alignment records, which inflates the plain `mapped` line. Always read `primary mapped` when you want to count reads.

So in barcode58, **only 7.9% of the reads are cassava**. That is lower than you might expect from a plant sample. These data do not look like a typical plant extract, and the other 92% of reads are something else. Remember that.

## Step 5: Keep the reads that did not map

We want the unmapped reads. `samtools view` selects reads from a BAM. `-f 4` means "keep only reads that **have** flag 4 set", and flag 4 means "unmapped". `-b` writes the output as a BAM file.

```
$ samtools view -b -f 4 mapping/barcode58_vs_cassava.sorted.bam > mapping/barcode58_unmapped.bam
```

Now turn those reads back into a FASTQ file, which the next tool needs:

```
$ samtools fastq mapping/barcode58_unmapped.bam > clean/barcode58_nonhost.fastq
samtools fastq: Coordinate sorted file.  Read pairs may be out of order
[M::bam2fq_mainloop] discarded 0 singletons
[M::bam2fq_mainloop] processed 8472 reads
```

The first message is a warning that applies to paired reads. Our reads are single, so you can **ignore it**. The last line is the useful one: **8,472 reads** processed, which is exactly the number of reads that did not map (9,196 − 724).

Compare the files before and after:

```
$ seqkit stats raw/barcode58.fastq clean/barcode58_nonhost.fastq
file                           format  type  num_seqs     sum_len  min_len  avg_len  max_len
raw/barcode58.fastq            FASTQ   DNA      9,196  13,567,398       13  1,475.4   43,663
clean/barcode58_nonhost.fastq  FASTQ   DNA      8,472  12,627,754       13  1,490.5   43,663
```

| | Before | After |
|---|---|---|
| Reads | 9,196 | **8,472** |
| Bases | 13,567,398 | 12,627,754 |
| Average length | 1,475 | 1,490 |

724 reads (7.9%) were removed. `clean/barcode58_nonhost.fastq` is the file that goes into the next lesson.

## The same thing in one command

As in Lesson 3.3, pipes let you skip the intermediate files. This does steps 3 to 5 together:

```
$ minimap2 -t 4 -ax map-ont reference/GCF_001659605.2_M.esculenta_v8_genomic.fna.gz raw/barcode58.fastq 2>/dev/null | samtools view -b -f 4 - | samtools fastq - > clean/barcode58_nonhost.fastq 2>/dev/null
```

We tested this against the step-by-step version, and it produced the same 8,472 reads. It is faster and leaves less on your disk, but you get no alignment file to inspect, so the step-by-step way is better for learning, and for keeping the `flagstat` numbers.

## Try it: barcode57

Repeat the whole procedure for **barcode57**. Change the file names, and run each step:

```
$ minimap2 -t 4 -ax map-ont reference/GCF_001659605.2_M.esculenta_v8_genomic.fna.gz raw/barcode57.fastq 2>/dev/null | samtools sort -o mapping/barcode57_vs_cassava.sorted.bam -
$ samtools flagstat mapping/barcode57_vs_cassava.sorted.bam | head -n 8
$ samtools view -b -f 4 mapping/barcode57_vs_cassava.sorted.bam > mapping/barcode57_unmapped.bam
$ samtools fastq mapping/barcode57_unmapped.bam > clean/barcode57_nonhost.fastq
$ seqkit stats raw/barcode57.fastq clean/barcode57_nonhost.fastq
```

You should find:

| | Value |
|---|---|
| `primary mapped` | **570** reads (6.65%) |
| Reads in `raw/barcode57.fastq` | 8,567 |
| Reads in `clean/barcode57_nonhost.fastq` | **7,997** |

Again, only about 7% of the reads are cassava.

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `Killed` | Not enough memory; Linux stopped the program | See Step 2: use the JupyterHub, or raise the WSL memory limit |
| `[E::main] failed to open file` | The reference file is not where you said | Check `ls -lh reference` and use Tab |
| `minimap2: command not found` | The environment is not active | `conda activate bioinfo` |
| It seems to hang | It is loading the genome; this takes 30 seconds or more | Wait. It is working |
| `[E::hts_open_format] Failed to open file` | The BAM was not written, so the mapping failed | Fix the mapping error first |
| `samtools fastq: Coordinate sorted file` warning | Normal for single reads | Ignore it |
| A slightly different count | A different tool version | Small differences are fine |
| `No space left on device` | Your disk is full | Free up space (you need about 1 GB for this lesson) |

## Practice check

1. Why do we remove host reads before looking for viruses?
2. What does `-x map-ont` tell `minimap2`?
3. `flagstat` shows `6570 mapped (43.68%)` and `724 primary mapped (7.87%)`. How many reads in barcode58 are cassava?
4. What do `-f 4` and `-b` do in `samtools view -b -f 4`?
5. You get the single word `Killed` while mapping. What happened?

### Answers

1. The host reads cannot tell us what else is in the sample, and in many samples they are most of the data and slow everything else down.
2. To use the settings for Oxford Nanopore reads.
3. 724 reads (7.87%). The plain `mapped` line counts alignment records, and long reads can have several.
4. `-f 4` keeps only the records flagged as unmapped, and `-b` writes the result as a BAM file.
5. The program ran out of memory and was stopped. Use the JupyterHub, or raise the memory limit for WSL.

## Summary of commands

| Command | What it does |
|---|---|
| `free -h` | Shows how much memory is available |
| `minimap2 -t 4 -ax map-ont ref.fna.gz reads.fastq \| samtools sort -o out.bam -` | Maps Nanopore reads and sorts them |
| `samtools flagstat file.bam` | Summarises the mapping (read `primary mapped`) |
| `samtools view -b -f 4 in.bam > unmapped.bam` | Keeps only unmapped reads |
| `samtools fastq in.bam > out.fastq` | Converts a BAM back to FASTQ |
| `seqkit stats files` | Compares the files before and after |
| `zcat file.gz \| grep ">" \| head` | Peeks at the sequence names in a compressed FASTA |

## What comes next

[Lesson 5.3: Classification with Kraken2](Lesson_3_Classification_with_Kraken2.md)
