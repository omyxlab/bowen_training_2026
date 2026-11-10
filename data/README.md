# Data for the Training

**Linux for Bioinformatics for Life Scientists · Bowen University · 16–18 November 2026**

This folder holds the large files that the lessons use but that are **not stored in the Git repository** (they are listed in `.gitignore`). Share them with participants by USB drive, a shared folder or the JupyterHub. Only this README and `SHA256SUMS.txt` are tracked.

The main training data (`training.tar.gz`, about 90 MB) are separate. They are published at <https://zenodo.org/records/24057548>.

## Contents

| Folder | File | Size | Used in | What it is |
|---|---|---|---|---|
| `cassava_reference/` | `GCF_001659605.2_M.esculenta_v8_genomic.fna.gz` | 180 MiB | Lesson 5.2 | The cassava (*Manihot esculenta*) reference genome from NCBI RefSeq, compressed. Do **not** unpack it; `minimap2` reads it directly |
| `kraken2_viral_database/` | `k2_viral_20261022.tar.gz` | 563 MiB | Lesson 5.3 | The Kraken2 viral database (RefSeq viruses, release of 22 October 2026). Unpacks to about 1.2 GB |
| `host_removal_results/barcode58/` | `barcode58_nonhost.fastq` | 24 MiB | Lesson 5.3 | The 8,472 barcode58 reads left **after** host removal |
| | `barcode58_vs_cassava.sorted.bam` and `.bai` | 12 MiB | Lesson 5.2 | The alignment of barcode58 to cassava, sorted and indexed |
| | `barcode58_unmapped.bam` | 10 MiB | Lesson 5.2 | The reads that did not map |
| `host_removal_results/barcode57/` | the same four files | | Lessons 5.2 and 5.3 | For barcode57: 7,997 reads after host removal |
| `ebola_references/` | `NC_002549.1.fasta` and `.gff3` | 19 KiB and 11 KiB | Lesson 3.1 | The Ebola reference genome and its annotation. A fallback if `efetch` does not work |
| | `KU143793.1.fasta` | 19 KiB | Capstone | The 2014 Makona reference genome |
| | `NC_001467.1.fasta` | 3 KiB | Capstone | African cassava mosaic virus DNA-A (the negative control) |

Total: about 833 MB.

## Using the files

### Cassava genome (Lesson 5.2)

```
$ cp cassava_reference/GCF_001659605.2_M.esculenta_v8_genomic.fna.gz ~/cassava_project/reference/
```

### Kraken2 database (Lesson 5.3)

```
$ mkdir -p ~/databases/kraken2_viral
$ tar -xzf kraken2_viral_database/k2_viral_20261022.tar.gz -C ~/databases/kraken2_viral
```

### If the session is running short of time

Host removal (Lesson 5.2) is the slowest step and the one that needs the most memory (about 5 GB). To skip it, give participants the **host-removed reads** and let them go straight to Lesson 5.3:

```
$ cp host_removal_results/barcode58/barcode58_nonhost.fastq ~/cassava_project/clean/
$ cp host_removal_results/barcode57/barcode57_nonhost.fastq ~/cassava_project/clean/
```

Participants who cannot map (for example, a laptop with too little memory) can still **read the alignments** with `samtools flagstat` on the supplied BAM files, so they see the same numbers as everyone else:

```
$ cp host_removal_results/barcode58/barcode58_vs_cassava.sorted.bam* ~/cassava_project/mapping/
$ samtools flagstat ~/cassava_project/mapping/barcode58_vs_cassava.sorted.bam | head -n 8
```

### Ebola references (Lesson 3.1, capstone)

```
$ cp ebola_references/NC_002549.1.fasta ebola_references/NC_002549.1.gff3 ~/ebola_project/reference/
```

## Checking the files

`SHA256SUMS.txt` lists a checksum for every file. After copying, run this **from inside this folder** to confirm nothing was damaged or cut short:

```
$ sha256sum -c SHA256SUMS.txt
```

Every line should end in `OK`. A truncated download is easy to miss, and the Kraken2 database in particular is large, so this check is worth doing.

## How these files were made

They were produced by running the lessons' commands on the course data with an environment built from the Module 0 command (minimap2 2.31, samtools 1.25, kraken2 2.17.2). The host-removal results are the output of Lesson 5.2 exactly as written. A different tool version can give slightly different read counts.
