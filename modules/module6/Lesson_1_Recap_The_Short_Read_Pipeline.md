# Lesson 6.1: Recap, The Short-Read Pipeline

**Module 6: Wrap-Up and Next Steps**

## What you will need

- Nothing new. This lesson summarises Modules 2 to 4.
- If you want to run it yourself: the `bioinfo` environment and the `Training` folder.

## What you will learn

By the end of this lesson you will be able to:

- Describe the short-read workflow as a sequence of steps, each with a purpose and a check
- Name the file that goes into and comes out of each step
- Run the whole workflow again, from an empty folder
- Recognise the common mistakes

## The workflow on one page

The Ebola analysis answered one question: *what is in this sample, and how does it differ from a known reference?* It was a chain of steps where each output became the next input.

```
raw reads (FASTQ)
    |  quality control          seqkit stats, fastqc
    v
    |  trimming                 fastp
    v
clean reads (FASTQ)  +  reference genome (FASTA, from efetch)
    |  mapping                  bwa index, bwa mem | samtools sort
    v
alignment (BAM)
    |  how well did it map?     samtools flagstat, coverage, tview
    |  calling variants         bcftools mpileup | bcftools call
    v
variants (VCF)  ---- which gene? ----  annotation (GFF3)
    |
    |  consensus                samtools consensus
    v
the sample's genome (FASTA)  ----  what is it?  ----  BLAST
```

| Step | Lesson | What goes in | What comes out | What you check |
|---|---|---|---|---|
| Set up the project | 3.1 | Accession | Folders, reference FASTA and GFF3 | File sizes: 19,335 and 11,506 bytes |
| Quality control | 3.2 | Raw FASTQ | Reports | Both files have 203,445 reads; FastQC results |
| Trimming | 3.2 | Raw FASTQ | Trimmed FASTQ | Pairs match; 168,889 pairs kept (83%); adapters gone |
| Mapping | 3.3 | Trimmed FASTQ, reference | Sorted, indexed BAM | `primary mapped`: 4,034 reads (1.19%) |
| Coverage | 3.4 | BAM | Numbers and `tview` | 98.97% covered, mean depth 17.2x |
| Variant calling | 4.1 | BAM, reference | VCF | 559 variants; 394 after the filter |
| Consensus | 4.2 | BAM | FASTA | 219 `N` of 18,959; header renamed |
| Interpretation | 4.3 | VCF, GFF3, consensus | Counts and BLAST | Variants in every gene; closest to 2014 Makona genomes |

## Every command, in order

This is the whole analysis. You can use it to repeat the workflow at home. It starts from an empty home folder with only `Training`.

**Environment and project (Lessons 2.2, 2.3, 3.1)**

```
conda activate bioinfo
cd ~
mkdir -p ebola_project/raw
cp Training/short_reads/paired/SRR1553607_1.fastq Training/short_reads/paired/SRR1553607_2.fastq ebola_project/raw/
cd ebola_project
mkdir -p reference qc trimmed mapping variants consensus results
efetch -db nuccore -id NC_002549.1 -format fasta > reference/NC_002549.1.fasta
efetch -db nuccore -id NC_002549.1 -format gff3 > reference/NC_002549.1.gff3
ls -l reference
```

**Quality control and trimming (Lesson 3.2)**

```
seqkit stats raw/*.fastq
fastqc --outdir qc --threads 2 raw/SRR1553607_1.fastq raw/SRR1553607_2.fastq
fastp -i raw/SRR1553607_1.fastq -I raw/SRR1553607_2.fastq -o trimmed/SRR1553607_1.trim.fastq -O trimmed/SRR1553607_2.trim.fastq -l 50 --detect_adapter_for_pe -h qc/fastp_report.html -j qc/fastp_report.json -w 4
seqkit stats trimmed/*.fastq
fastqc --outdir qc --threads 2 --quiet trimmed/SRR1553607_1.trim.fastq trimmed/SRR1553607_2.trim.fastq
```

**Mapping and coverage (Lessons 3.3 and 3.4)**

```
bwa index reference/NC_002549.1.fasta
bwa mem -t 4 reference/NC_002549.1.fasta trimmed/SRR1553607_1.trim.fastq trimmed/SRR1553607_2.trim.fastq 2>/dev/null | samtools sort -o mapping/SRR1553607.sorted.bam -
samtools index mapping/SRR1553607.sorted.bam
samtools flagstat mapping/SRR1553607.sorted.bam
samtools coverage mapping/SRR1553607.sorted.bam
samtools tview -d T -p NC_002549.1:285 mapping/SRR1553607.sorted.bam reference/NC_002549.1.fasta | head -n 9
```

**Variants (Lesson 4.1)**

```
bcftools mpileup -f reference/NC_002549.1.fasta mapping/SRR1553607.sorted.bam | bcftools call -mv --ploidy 1 -o variants/SRR1553607.vcf
grep -vc "^#" variants/SRR1553607.vcf
bcftools view -i "QUAL>=30 && INFO/DP>=10" variants/SRR1553607.vcf -o variants/SRR1553607.filtered.vcf
grep -vc "^#" variants/SRR1553607.filtered.vcf
```

**Consensus (Lesson 4.2)**

```
samtools consensus -a -f fasta mapping/SRR1553607.sorted.bam -o consensus/SRR1553607.consensus.fasta
grep -v ">" consensus/SRR1553607.consensus.fasta | grep -o "N" | wc -l
nano consensus/SRR1553607.consensus.fasta
cp consensus/SRR1553607.consensus.fasta results/
```

(In `nano`, change the first line to `>SRR1553607_consensus`.)

**Interpretation (Lesson 4.3)**

```
bcftools view -H -t NC_002549.1:5900-8305 variants/SRR1553607.filtered.vcf | wc -l
```

Then repeat for each gene and use NCBI BLAST on the consensus. The run takes about a minute of computing time in total. The time you spend is in understanding each step.

## The ideas that tie it together

- **Look at the data first.** QC comes before everything. Every later step assumes the data are sound.
- **Read the right number.** `primary mapped`, not `mapped`. `grep -o "N" | wc -l`, not `grep -c "N"`. Lines ÷ 4, not a count of `@`.
- **A filter is a decision.** Trimming removed 17% of the pairs; filtering removed 165 variants. State what you did and why.
- **The reference defines the result.** 559 variants against a 1976 genome. What would a different reference give? You will test that in the capstone.
- **Missing data are shown honestly.** An `N` means "we do not know".
- **Only 1.2% of the reads were the virus.** That is normal in a clinical sample, and it did not stop us getting a good genome.

## Common mistakes

| Mistake | What goes wrong | What to do |
|---|---|---|
| Forgetting `conda activate bioinfo` | Tools say "command not found" | Check that the prompt starts with `(bioinfo)` |
| Working in the wrong folder | "No such file or directory" | `pwd`, then `cd ~/ebola_project` |
| Skipping `samtools index` | Region queries and `tview` fail | Index every sorted BAM |
| Reading `mapped` instead of `primary mapped` | Wrong percentage | Use the primary line |
| Counting `N` with `grep -c` | Counts lines, not letters | `grep -o "N" \| wc -l` |
| Mixing files from different references | Results that do not match | Name files after the reference |
| Editing the raw files | The originals are lost | Always work on copies |
| Using `rm -r` carelessly | No undo | Check with `ls` first |

## Check your understanding

1. In one sentence each, what do `fastp`, `bwa mem`, `bcftools call` and `samtools consensus` do?
2. Which file format comes out of mapping, and which out of variant calling?
3. Why did only 1.19% of the reads map, and was that a problem?
4. Why does the number of variants depend on the reference?

### Answers

1. `fastp` removes adapters and low-quality reads. `bwa mem` finds where each read fits on the reference. `bcftools call` decides which positions differ from the reference. `samtools consensus` writes the sample's own sequence.
2. A sorted BAM comes out of mapping, and a VCF out of variant calling.
3. Because the sample is a mixture, and most reads came from other sources, not the virus. It was not a problem: the virus reads still covered 99% of the genome at 17x.
4. Variants are differences from the reference. A distant reference gives many differences, and a close one gives few.

## What comes next

[Lesson 6.2: Recap, the long-read pipeline](Lesson_2_Recap_The_Long_Read_Pipeline.md)
