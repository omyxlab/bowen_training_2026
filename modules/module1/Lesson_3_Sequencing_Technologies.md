# Lesson 1.3: Sequencing Technologies

**Module 1: Welcome and Introduction to Bioinformatics**

## What you will learn

By the end of this lesson you will be able to:

- Explain what DNA sequencing does
- Describe the three broad generations of sequencing technology
- Compare Illumina, Oxford Nanopore and PacBio sequencing
- Explain the difference between short and long reads, and between single-end and paired-end reads
- Choose a suitable technology for a given question
- Say which technology produced each dataset in this course

## What is sequencing?

**Sequencing** means determining the order of the bases (A, C, G and T in DNA; A, C, G and U in RNA) in a molecule. The result is a text string of letters called a **read**.

No machine reads a whole genome end to end in one go. Instead, the DNA is broken or selected into fragments and each fragment is read separately. The analysis later puts the information back together. How long each fragment is, how accurately it is read, and how many fragments are read at once are what separate the technologies.

## Three generations of sequencing

### First generation: Sanger sequencing

- Developed by Frederick Sanger in the 1970s.
- Reads **one fragment at a time**, typically up to about 800 to 1,000 bases.
- Very accurate.
- Slow and expensive for large projects, but still widely used to **confirm** a specific result, such as checking a gene or a single variant.

### Second generation: "next-generation" sequencing (Illumina)

- Became available in the mid-2000s and transformed biology.
- Reads **millions to billions of short fragments in parallel**.
- **Short reads**, typically 50 to 300 bases.
- Very accurate (a small fraction of a percent of bases are wrong) and low cost per base.
- The most widely used technology today.

**How it works, in brief.** The DNA fragments are attached to a glass surface and copied many times so that each fragment forms a small cluster. The machine then adds fluorescently labelled bases one cycle at a time and takes a picture after each cycle. The colour in each picture tells it which base was added. This is called *sequencing by synthesis*.

### Third generation: long-read sequencing

These technologies read **single, much longer molecules**.

- **Oxford Nanopore** passes a DNA or RNA molecule through a tiny protein pore. As each base passes through, it changes an electrical current in a characteristic way, and software (called *basecalling*) turns those changes into letters. Reads can range from a few dozen bases to hundreds of thousands. The devices can be small and portable, so they can be used in the field or in a hospital laboratory.
- **PacBio** reads single molecules using fluorescence in tiny wells. Its high-accuracy mode ("HiFi") produces reads typically around 10 to 25 thousand bases long with very high accuracy.

## Comparison

| | Sanger | Illumina | Oxford Nanopore | PacBio |
|---|---|---|---|---|
| **Read length (typical)** | up to about 1,000 bases | 50 to 300 bases | a few hundred to many thousands of bases | about 10 to 25 thousand bases (HiFi) |
| **Reads per run** | one at a time | millions to billions | thousands to millions | hundreds of thousands to millions |
| **Accuracy** | very high | very high | lower than Illumina; improving with each new chemistry | very high (HiFi) |
| **Typical error type** | rare | mostly single-base changes | more insertions and deletions | rare in HiFi |
| **Main strength** | confirming results | accuracy and low cost per base | long reads; portable; real-time | long and accurate reads |
| **Main limitation** | low throughput | short reads struggle with repeats | higher error rate | cost and equipment |
| **In this course** | not used | Ebola virus data | cassava data | not used |

These numbers are typical values, not exact ones. They differ between instruments and kits and change as the technologies improve.

## Short reads and long reads

Imagine you want to put a page of text back together after it has been cut into pieces.

- With **short strips**, each piece has only a few words. If a phrase appears in several places on the page, you cannot tell which piece belongs where.
- With **long strips**, each piece spans enough text to be placed with confidence.

That is the key difference:

- **Short reads** are accurate and cheap, and are excellent for counting and for finding small changes (such as single-base variants) when a good reference exists.
- **Long reads** can span repeated regions and large rearrangements, and can cover a whole small genome, or a whole gene, in one read. They are very useful when no good reference exists, or when you need fast results with portable equipment.

Neither is "better". The right choice depends on the question.

## Single-end and paired-end reads

Illumina fragments can be read from **one end** (*single-end*) or from **both ends** (*paired-end*).

```
  DNA fragment:   5'---------------------------------------------3'
  Read 1 (forward)  ------>                              
  Read 2 (reverse)                              <------
```

Paired-end sequencing gives you two reads from every fragment, and you know roughly how far apart they are. This helps place reads correctly in the genome.

Paired reads are stored in **two files**, usually named `_1` and `_2` (or `_R1` and `_R2`). The first read in file 1 and the first read in file 2 come from the same fragment, and the files are in the same order.

The Ebola data in this course are paired-end: `SRR1553607_1.fastq` and `SRR1553607_2.fastq`.

## Quality and errors

Every sequencing technology makes mistakes, and every base in the output comes with a **quality score** that estimates how likely it is to be wrong. Quality scores are explained in Lesson 1.4.

Different technologies make different kinds of mistakes. Illumina mostly makes occasional single-base errors. Nanopore more often gets the number of repeated bases wrong, for example reading `AAAA` as `AAA`. This is one reason why we use different tools and settings for short and long reads.

## Which technology for which question?

| Question | A sensible choice | Why |
|---|---|---|
| Find single-base differences between many samples and a reference | Illumina | Accurate, low cost per sample |
| Count the microbes in a gut sample using a marker gene (16S) | Illumina | Short amplicons suit short reads |
| Sequence a whole plant virus genome quickly, in the field | Nanopore | Portable, long reads, fast |
| Assemble a genome with many repeats | PacBio HiFi or Nanopore | Long reads span the repeats |
| Check that one gene sequence is correct | Sanger | Simple, accurate, single fragment |

## The technologies behind our data

| Dataset | Technology | Evidence in the file |
|---|---|---|
| **Ebola virus** `SRR1553607` | **Illumina HiSeq 2500**, paired-end, RNA-Seq | Reads are all exactly **101 bases** long: 203,445 in each of the two files |
| **Cassava** `barcode57`, `barcode58`, `sample3` | **Oxford Nanopore** | Read lengths vary widely (see below), and the header carries a tag naming the `r10.4.1` flow cell chemistry |

You can see the difference in the read lengths:

| File | Number of reads | Shortest | Average | Longest |
|---|---|---|---|---|
| `SRR1553607_1.fastq` (Illumina) | 203,445 | 101 | 101 | 101 |
| `barcode57.fastq` (Nanopore) | 8,567 | 3 | 1,421 | 34,441 |
| `barcode58.fastq` (Nanopore) | 9,196 | 13 | 1,475 | 43,663 |

Every Illumina read is the same length because the machine runs a fixed number of cycles. Nanopore reads are as long as the molecules that happened to pass through the pores, so their lengths vary a lot. You will compute these numbers yourself later in the course.

The names `barcode57` and `barcode58` come from **barcoding**: short tags added to each sample so that several samples can be sequenced together on one flow cell and separated again afterwards.

## Key terms

| Term | Meaning |
|---|---|
| **Base** | One letter of DNA or RNA (A, C, G, T or U) |
| **Read** | The sequence obtained from one fragment |
| **Short read / long read** | Reads of roughly hundreds of bases / thousands of bases or more |
| **Paired-end** | Both ends of each fragment are read, giving two files |
| **Library preparation** | Getting DNA or RNA ready for sequencing |
| **Flow cell** | The surface or device where sequencing takes place |
| **Basecalling** | Converting the machine's raw signal into letters |
| **Barcode** | A tag used to tell samples apart when they are sequenced together |
| **Throughput** | How much sequence a run produces |

## Check your understanding

1. Why does an Illumina file have reads that are all the same length?
2. Give one advantage of long reads and one advantage of short reads.
3. How do you recognise paired-end data from file names alone?
4. Which technology produced the cassava data? What was the clue?
5. You need to sequence a plant virus quickly at a field station with no laboratory. Which technology would you consider first, and why?

### Answers

1. The machine runs a fixed number of cycles, and each cycle reads one base.
2. Long reads: they can span repeats and cover whole small genomes. Short reads: they are accurate and low in cost.
3. There are two files per sample with matching names ending in `_1` and `_2` (or `_R1` and `_R2`).
4. Oxford Nanopore. The reads vary greatly in length, and the headers mention the Nanopore flow cell chemistry.
5. Nanopore: the device is portable and the reads are long and available quickly.

## What comes next

[Lesson 1.4: Biological file formats](Lesson_4_Biological_File_Formats.md)
