# Lesson 1.5: Biological Databases

**Module 1: Welcome and Introduction to Bioinformatics**

## What you will learn

By the end of this lesson you will be able to:

- Explain why public biological databases matter
- Say what NCBI, ENA, Ensembl and UniProt are for
- Tell a raw-data archive from a curated reference database
- Read an accession number and know what kind of record it points to
- Look up the Ebola sequencing run and reference genome used in this course

**You do not need a terminal for this lesson.** You will use a web browser.

## Why public databases?

Scientists are expected to deposit the data behind published work in public databases, so that anyone can check and reuse it. Everything in this course comes from them:

- The Ebola reads were downloaded from a public sequence archive.
- The reference genome and its gene annotation come from NCBI.
- The tools you will use search these same databases.

Knowing where biological data live, and how to find the right record, is a basic skill.

## The big three sequence archives

Three organisations around the world store public nucleotide sequences. They **exchange data with each other**, so a sequence submitted to one becomes available in all three.

| Organisation | Based at | Website |
|---|---|---|
| **NCBI** (National Center for Biotechnology Information) | USA | <https://www.ncbi.nlm.nih.gov/> |
| **ENA** (European Nucleotide Archive, run by EMBL-EBI) | Europe | <https://www.ebi.ac.uk/ena/> |
| **DDBJ** (DNA Data Bank of Japan) | Japan | <https://www.ddbj.nig.ac.jp/> |

They hold two broad kinds of data:

- **Raw reads** (FASTQ), stored in the **Sequence Read Archive (SRA)** at NCBI, and in ENA and DDBJ.
- **Assembled sequences** and genomes, stored in **GenBank** (NCBI) and its counterparts.

## NCBI

NCBI is the database you will use most. It hosts many connected resources:

| Resource | What it holds |
|---|---|
| **Nucleotide** | DNA and RNA sequences (GenBank and RefSeq records) |
| **SRA** | Raw sequencing reads |
| **BioProject and BioSample** | Descriptions of the study and the sample |
| **Taxonomy** | The classification of organisms |
| **Gene** | Gene-focused information |
| **Protein** | Protein sequences |
| **BLAST** | A tool for finding similar sequences (<https://blast.ncbi.nlm.nih.gov/>) |
| **PubMed** | Scientific literature |

### GenBank and RefSeq

- **GenBank** holds sequences exactly as submitted. Anyone can submit, so there can be several records for the same gene or genome, of differing quality.
- **RefSeq** is a smaller collection that NCBI **curates**: one reviewed, well-annotated record per organism or gene. It is the standard choice for a reference.

The reference genome we use in this course is a RefSeq record: **`NC_002549.1`**, the Ebola virus (*Zaire ebolavirus*) genome. It is the Yambuku-Mayinga isolate from 1976 (note "COD/1976" in its name: the Democratic Republic of the Congo, 1976). It is a different virus isolate from the 2014 patient sample, which is why we expect to find differences between them.

## ENA

The European Nucleotide Archive contains the same public sequence data as NCBI, with its own search interface. It is especially convenient for **downloading raw reads**: each run page gives direct links to FASTQ files. The 16S microbiome files in your training folder were downloaded from ENA with the script that came with them.

## Ensembl

**Ensembl** (<https://www.ensembl.org/>) provides genomes together with their annotation (genes, transcripts, variants) and a genome browser. It is best known for vertebrates, including human and many model animals. Related sites cover other groups such as plants, fungi and bacteria.

## UniProt

**UniProt** (<https://www.uniprot.org/>) is the main database of **protein sequences and their functions**. It has two parts:

- **Swiss-Prot**: entries reviewed by experts.
- **TrEMBL**: automatically annotated entries that nobody has reviewed yet.

Its entries tell you what a protein does, where it is found, and its structure and domains. For example, four of the Ebola virus proteins in Swiss-Prot (strain Mayinga-76):

| Protein | UniProt accession | What it is |
|---|---|---|
| NP | `P18272` | Nucleoprotein |
| VP40 | `Q05128` | Matrix protein |
| GP | `Q05320` | Envelope glycoprotein |
| L | `Q05318` | RNA-directed RNA polymerase |

## Other databases worth knowing

| Database | What it holds |
|---|---|
| **PDB** (Protein Data Bank) | Three-dimensional protein structures |
| **InterPro / Pfam** | Protein families and domains |
| **KEGG** | Metabolic and signalling pathways |
| **GEO** | Gene expression datasets |
| **SILVA** | Ribosomal RNA sequences, used to identify microbes from 16S data |

## Reading an accession number

Every record has an **accession**: a unique code that identifies it. The first letters tell you what kind of record it is.

| Accession looks like | What it is | Example |
|---|---|---|
| `SRR`, `ERR`, `DRR` + digits | A sequencing **run** (the actual reads) | `SRR1553607` |
| `SRX`, `ERX`, `DRX` | An **experiment** (the library and method) | |
| `SRS`, `ERS`, `DRS` | A **sample** in the read archives | |
| `SRP`, `ERP`, `DRP` | A **study** in the read archives | |
| `PRJNA`, `PRJEB`, `PRJDB` + digits | A **BioProject** (the overall study) | `PRJNA257197` |
| `SAMN`, `SAMEA` + digits | A **BioSample** (the biological sample) | `SAMN02952048` |
| `NC_` + digits | A RefSeq **complete genomic molecule** | `NC_002549.1` |
| `NM_`, `NP_` | A RefSeq mRNA, and a RefSeq protein | |
| `GCF_` or `GCA_` + digits | A genome **assembly** (`GCF` is RefSeq, `GCA` is GenBank) | |
| Six characters starting with a letter, such as `P18272` | A **UniProt** protein entry | `P18272` |
| `ENSG`, `ENST`, `ENSP` + digits | Ensembl gene, transcript and protein | |

### Always note the version

Many accessions end in a **version number** after a dot, such as `NC_002549.1`. If the sequence is ever corrected, the version number goes up (`.2`). To make sure someone else uses exactly the same sequence as you, **always record the full accession with its version**.

## Try it: look up the Ebola run

This is a short exercise in your browser. Check what you find against the values given.

### 1. The sequencing run

1. Open <https://www.ebi.ac.uk/ena/browser/view/SRR1553607>.
2. Read the entry and check these details:

| Field | You should find |
|---|---|
| Scientific name | *Zaire ebolavirus* |
| Study | `PRJNA257197` ("Zaire ebolavirus Genome sequencing") |
| Sample | `SAMN02952048` (sample name `NM042.2`) |
| Instrument | Illumina HiSeq 2500 |
| Library layout | PAIRED |
| Library strategy | RNA-Seq |
| Read count | 203,445 |
| Country and collection date | Sierra Leone, 2014-06-09 |

You can find the same run at NCBI by opening <https://www.ncbi.nlm.nih.gov/sra/SRR1553607>. NCBI's own description of the project is that it is "Zaire ebolavirus sample sequencing from the 2014 outbreak in Sierra Leone, West Africa."

### 2. The reference genome

1. Open <https://www.ncbi.nlm.nih.gov/nuccore/NC_002549.1>.
2. Find the **definition line**: "Zaire ebolavirus isolate Ebola virus/H.sapiens-tc/COD/1976/Yambuku-Mayinga, complete genome".
3. Notice that the length is **18,959 bp**.
4. Scroll down to the **Features** section and find the seven genes: NP, VP35, VP40, GP, VP30, VP24 and L.

This is the record from which the reference FASTA and GFF3 file used in the course were taken.

### 3. A protein record

1. Open <https://www.uniprot.org/> and search for `P18272`.
2. You should find the Ebola virus nucleoprotein (NP), a reviewed (Swiss-Prot) entry.

## Key terms

| Term | Meaning |
|---|---|
| **Accession** | A unique code identifying a database record |
| **INSDC** | The partnership of NCBI, ENA and DDBJ that share public sequence data |
| **SRA** | The Sequence Read Archive, where raw reads are stored |
| **GenBank** | NCBI's collection of submitted sequences |
| **RefSeq** | NCBI's curated collection of reference sequences |
| **Curated** | Reviewed and corrected by experts |
| **Swiss-Prot / TrEMBL** | The reviewed and unreviewed parts of UniProt |
| **BioProject / BioSample** | A record of the study / of the biological sample |

## Check your understanding

1. What kind of record is `SRR1553607`? What kind is `NC_002549.1`?
2. What is the difference between GenBank and RefSeq?
3. Where would you look for the function of a protein?
4. Why should you record the version number of an accession?
5. In what country and year was the Ebola sample collected?

### Answers

1. `SRR1553607` is a sequencing run (raw reads). `NC_002549.1` is a RefSeq complete genomic sequence.
2. GenBank holds submitted sequences as they were received. RefSeq is curated: reviewed, non-redundant reference records.
3. UniProt.
4. The sequence can be corrected, which increases the version number; recording it ensures others can use exactly the same sequence.
5. Sierra Leone, in 2014 (collected 9 June 2014).

## What comes next

You have now finished the background for the course. In **Module 2** you open the terminal for the first time.
