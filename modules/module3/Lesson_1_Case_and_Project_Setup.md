# Lesson 3.1: Case and Project Setup

**Module 3: Short-Read Analysis I**

## What you will need

- The `bioinfo` environment active (your prompt begins with `(bioinfo)`)
- `~/ebola_project/raw/` holding `SRR1553607_1.fastq` and `SRR1553607_2.fastq` (from Lesson 2.3)
- An internet connection

## What you will learn

By the end of this lesson you will be able to:

- State the question this analysis is trying to answer
- Lay out a project folder so every kind of file has a home
- Download a reference genome and its annotation from NCBI with `efetch`
- Check that a download is complete

## The case

Here is the situation we are working with. A patient sample, collected in Sierra Leone in June 2014 during the Ebola outbreak there, was sequenced on an Illumina machine. The data are public. You have the reads. The questions are:

> **Is Ebola virus in this sample? How much of the genome do we recover? And how does this virus differ from a known reference genome?**

To answer them we compare the reads with a reference genome: a trusted sequence of the same virus. The steps, which you will follow over the next modules, are:

```
raw reads -> quality control -> trimming -> mapping -> coverage -> variants -> consensus genome
```

## Why organise a project?

By the end you will have raw reads, trimmed reads, reports, alignments, variant files and a final genome. Mixed together in one folder, they are impossible to follow. We give every kind of file its own folder, and we never touch the raw data.

## Step 1: Make the folders

You already have `ebola_project/raw`. Add the rest, using `mkdir -p` to make several at once:

```
$ cd ~/ebola_project
$ ls
raw
$ mkdir -p reference qc trimmed mapping variants consensus results
$ ls
consensus  mapping  qc  raw  reference  results  trimmed  variants
```

| Folder | What goes in it |
|---|---|
| `raw` | The original reads. **Never edited** |
| `reference` | The reference genome and its annotation |
| `qc` | Quality reports |
| `trimmed` | Cleaned reads |
| `mapping` | Alignments (BAM files) |
| `variants` | Variant files (VCF) |
| `consensus` | The sample's own genome sequence |
| `results` | Final results you want to keep |

If `raw` is missing or empty, go back to Lesson 2.3 and copy the reads in.

## Step 2: Download the reference genome

We use the NCBI RefSeq record **`NC_002549.1`**, the Ebola virus genome you met in Lesson 1.5. The tool `efetch` (part of NCBI's Entrez Direct) downloads records by accession.

Download the sequence in FASTA format:

```
$ efetch -db nuccore -id NC_002549.1 -format fasta > reference/NC_002549.1.fasta
```

| Part | Meaning |
|---|---|
| `efetch` | The download program |
| `-db nuccore` | Look in the Nucleotide database |
| `-id NC_002549.1` | The accession (with its version) |
| `-format fasta` | The file format we want |
| `> reference/NC_002549.1.fasta` | Save the result in a file (Lesson 2.4) |

Now the **annotation**: the same genome as a GFF3 file, listing where the genes are.

```
$ efetch -db nuccore -id NC_002549.1 -format gff3 > reference/NC_002549.1.gff3
```

### If you see curl errors

`efetch` may print a few lines of red or plain text that look like this:

```
curl: (56) OpenSSL SSL_read: ... unexpected eof while reading
 ERROR:  curl command failed ... with: 56
```

and then still finish with `HTTP/1.0 200 OK`. This is a temporary connection hiccup. **`efetch` retries by itself**, so the download usually succeeds anyway. The test is whether the files are the right size, as in the next step.

## Step 3: Check the downloads

```
$ ls -l reference
total 32
-rw-r--r-- 1 yourname yourname 19335 Nov 10 23:23 NC_002549.1.fasta
-rw-r--r-- 1 yourname yourname 11506 Nov 10 23:23 NC_002549.1.gff3
```

The FASTA should be **19,335 bytes** and the GFF3 **11,506 bytes**. If either shows `0`, the download failed; run that `efetch` command again.

Look at the start of the FASTA, and count the sequences:

```
$ head -n 3 reference/NC_002549.1.fasta
>NC_002549.1 Zaire ebolavirus isolate Ebola virus/H.sapiens-tc/COD/1976/Yambuku-Mayinga, complete genome
CGGACACACAAAAAGAAAGAAGAATTTTTAGGATCTTTTGTGTGCGAATAACTATGAGGAAGATTAATAA
TTTTCCTCTCATTGAAATTTATATCGGAATTTAAATTGAAATTGTTACTGTAATCACACCTGGTTTGTTT
$ grep -c ">" reference/NC_002549.1.fasta
1
```

One header line means one sequence: the single genome. Check its length with `seqkit stats`:

```
$ seqkit stats reference/NC_002549.1.fasta
file                         format  type  num_seqs  sum_len  min_len  avg_len  max_len
reference/NC_002549.1.fasta  FASTA   DNA          1   18,959   18,959   18,959   18,959
```

The genome is **18,959 bases** long, as you saw on the NCBI page in Lesson 1.5.

## Step 4: Look at the annotation

```
$ head -n 8 reference/NC_002549.1.gff3
##gff-version 3
#!gff-spec-version 1.21
#!processor NCBI annotwriter
##sequence-region NC_002549.1 1 18959
##species https://www.ncbi.nlm.nih.gov/Taxonomy/Browser/wwwtax.cgi?id=186538
NC_002549.1	RefSeq	region	1	18959	.	+	.	ID=NC_002549.1:1..18959;Dbxref=taxon:186538;gbkey=Src;genome=genomic;isolate=Ebola virus/H.sapiens-tc/COD/1976/Yambuku-Mayinga;mol_type=viral cRNA
NC_002549.1	RefSeq	five_prime_UTR	1	55	.	+	.	ID=id-NC_002549.1:1..55;Note=leader region;function=regulation or initiation of RNA replication;gbkey=5'UTR
NC_002549.1	RefSeq	gene	56	3026	.	+	.	ID=gene-ZEBOVgp1;Dbxref=GeneID:911830;Name=NP;gbkey=Gene;gene=NP;gene_biotype=protein_coding;locus_tag=ZEBOVgp1
```

Lines beginning with `#` are comments. The rest are **features**: the whole region, the 5' leader, and the first gene, **NP**, which spans positions 56 to 3,026 (Lesson 1.4).

The file has several kinds of feature (genes, coding sequences, regulatory regions). To see only the **genes**, search for the pattern that appears in a gene's ID:

```
$ grep "ID=gene-" reference/NC_002549.1.gff3
NC_002549.1	RefSeq	gene	56	3026	.	+	.	ID=gene-ZEBOVgp1;Dbxref=GeneID:911830;Name=NP;gbkey=Gene;gene=NP;gene_biotype=protein_coding;locus_tag=ZEBOVgp1
NC_002549.1	RefSeq	gene	3032	4407	.	+	.	ID=gene-ZEBOVgp2;Dbxref=GeneID:911827;Name=VP35;gbkey=Gene;gene=VP35;gene_biotype=protein_coding;locus_tag=ZEBOVgp2
NC_002549.1	RefSeq	gene	4390	5894	.	+	.	ID=gene-ZEBOVgp3;Dbxref=GeneID:911825;Name=VP40;gbkey=Gene;gene=VP40;gene_biotype=protein_coding;locus_tag=ZEBOVgp3
NC_002549.1	RefSeq	gene	5900	8305	.	+	.	ID=gene-ZEBOVgp4;Dbxref=GeneID:911829;Name=GP;gbkey=Gene;gene=GP;gene_biotype=protein_coding;locus_tag=ZEBOVgp4
NC_002549.1	RefSeq	gene	8288	9740	.	+	.	ID=gene-ZEBOVgp5;Dbxref=GeneID:911826;Name=VP30;gbkey=Gene;gene=VP30;gene_biotype=protein_coding;locus_tag=ZEBOVgp5
NC_002549.1	RefSeq	gene	9885	11518	.	+	.	ID=gene-ZEBOVgp6;Dbxref=GeneID:911828;Name=VP24;Note=putative;gbkey=Gene;gene=VP24;gene_biotype=protein_coding;locus_tag=ZEBOVgp6
NC_002549.1	RefSeq	gene	11501	18282	.	+	.	ID=gene-ZEBOVgp7;Dbxref=GeneID:911824;Name=L;gbkey=Gene;gene=L;gene_biotype=protein_coding;locus_tag=ZEBOVgp7
$ grep -c "ID=gene-" reference/NC_002549.1.gff3
7
```

Seven genes: **NP, VP35, VP40, GP, VP30, VP24 and L**. You will need these coordinates in Module 4.

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `efetch: command not found` | The environment is not active | `conda activate bioinfo` |
| A file is 0 bytes, or much smaller than expected | The download failed | Run the `efetch` command again |
| `ERROR: curl command failed` repeatedly, and no file | A network problem | Check your connection; try again later or ask a trainer for the two files |
| `mkdir: cannot create directory 'qc': File exists` | The folder is already there | Nothing to fix |
| `No such file or directory` for `reference/...` | You are in the wrong folder | `cd ~/ebola_project` |

If `efetch` will not work at all, you can download the files in a web browser instead. On the NCBI page for `NC_002549.1` choose **Send to**, then **Complete Record**, then **File**, and pick the format (FASTA, or GFF3). Then ask a trainer how to move the files into the `reference` folder.

## Practice check

1. Why do we keep `raw` separate from everything else?
2. How long is the Ebola virus genome, and where did you see that?
3. What does `grep "ID=gene-"` do?
4. Which gene starts at position 5900?

### Answers

1. So the original data can never be changed by accident, and we can always start again from them.
2. 18,959 bases: in the FASTA with `seqkit stats`, and on the NCBI page and the GFF3 header (`##sequence-region NC_002549.1 1 18959`).
3. Prints only the lines that contain `ID=gene-`, which are the gene features.
4. GP.

## Summary of commands

| Command | What it does |
|---|---|
| `mkdir -p a b c` | Makes several folders at once |
| `efetch -db nuccore -id ACC -format fasta` | Downloads a sequence in FASTA format |
| `efetch -db nuccore -id ACC -format gff3` | Downloads the annotation in GFF3 format |
| `ls -l` | Checks file sizes |
| `seqkit stats file` | Summarises a sequence file |
| `grep -c ">" file` | Counts the sequences in a FASTA file |

## What comes next

[Lesson 3.2: Quality control and trimming](Lesson_2_Quality_Control_and_Trimming.md)
