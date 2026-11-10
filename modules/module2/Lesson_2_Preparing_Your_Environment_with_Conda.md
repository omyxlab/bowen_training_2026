# Lesson 2.2: Preparing Your Environment with Conda

**Module 2: Working in the Linux Terminal**

## What you will need

- A terminal open
- Miniforge installed (from [Module 0](../module0/Getting_Started.md))
- An internet connection, **if** you still need to build the environment

## What you will learn

By the end of this lesson you will be able to:

- Explain what Conda, a channel and an environment are
- Check whether the `bioinfo` environment already exists
- Create the `bioinfo` environment, if it does not
- Activate and deactivate an environment, and tell which one is active
- Confirm that every tool for the course is installed and working

## Why this matters

Bioinformatics programs are rarely a single file you can download and double-click. Each depends on other programs and libraries, often at particular versions, and installing them by hand is slow and fragile.

**Conda** is a tool that does this for you. You ask for a program by name, and Conda finds it, together with everything it needs, and installs them in a self-contained folder. That folder is called an **environment**. We put all the tools for this course into one environment called `bioinfo`, so they never interfere with anything else on your computer.

## Key ideas

| Term | Meaning |
|---|---|
| **Conda** | A package and environment manager. It installs software and keeps versions organised |
| **Package** | One piece of software (for example `samtools`) together with its details |
| **Environment** | A separate box of packages. You can have many, and switch between them |
| **Channel** | A place packages are downloaded from |
| **`base`** | The default environment that comes with Miniforge. **Do not install your tools here** |
| **`conda-forge`** | A large community channel of general software |
| **`bioconda`** | A channel dedicated to bioinformatics software |

Why keep tools out of `base`? If everything goes into one place, different programs eventually demand different versions of the same library and break each other. One environment per project keeps things predictable, and if something goes wrong you can delete that environment and start again.

## Step 1: Is Conda working?

```
$ conda --version
conda 24.7.1
```

Your number may be different; any version is fine. If you see `conda: command not found`, return to Part 6 of the [Getting Started guide](../module0/Getting_Started.md), then come back.

Now see which environments exist on your computer:

```
$ conda env list
# conda environments:
#
base                  *  /home/yourname/miniforge3
```

The `*` marks the environment you are using right now. Your prompt probably begins with `(base)`.

**Now look at your list.**

- If you see a line starting with **`bioinfo`**, you built the environment before the course. Go to **Path A: Verify**.
- If you see only `base` (and maybe others, but not `bioinfo`), you need to create it. Go to **Path B: Create**.
- If you are on the **JupyterHub**, the environment is already built. Follow **Path A**.

## Path A: Verify the environment you already have

If you have a `bioinfo` environment, the output looks like this:

```
$ conda env list
# conda environments:
#
base                  *  /home/yourname/miniforge3
bioinfo                  /home/yourname/miniforge3/envs/bioinfo
```

Switch it on:

```
$ conda activate bioinfo
```

Your prompt changes, and now begins with `(bioinfo)`:

```
(bioinfo) yourname@YOUR-PC:~$
```

Then skip ahead to **Step 4: Check that every tool works**. If a tool is missing, use Path B to fix it.

## Path B: Create the environment

### Step 2: One command builds everything

Copy this command (it is one long line), paste it into the terminal, and press **Enter**:

```
conda create -n bioinfo -c conda-forge -c bioconda seqkit fastqc fastp bwa minimap2 samtools bcftools nanoplot entrez-direct kraken2 krona krakentools -y
```

Here is what each part does:

| Part | Meaning |
|---|---|
| `conda create` | Make a new environment |
| `-n bioinfo` | Name it `bioinfo` |
| `-c conda-forge -c bioconda` | Look for packages in these two channels |
| `seqkit ... krakentools` | The 12 tools we need (you will meet each one during the course) |
| `-y` | Say "yes" to Conda's confirmation question automatically |

### What to expect

- Conda first spends a short while "solving the environment": working out which versions fit together.
- It then lists a plan and downloads the packages. **The plan has around 150 packages and about 510 MB of downloads.** The finished environment takes up about 2.7 GB on disk.
- Pages of text scroll past. This is normal.
- It finishes with `Preparing transaction: done`, `Verifying transaction: done` and `Executing transaction: done`, then your prompt returns.

How long it takes depends almost entirely on your internet connection: a few minutes on a fast one, up to half an hour on a slow one. **This is the part that can run in the background**: start it, then take a break. Do not close the terminal until the prompt returns.

### A message you can ignore

Near the end, Krona prints a message that begins "Krona installed. You still need to manually update the taxonomy databases...". **You can ignore this.** The Krona tool we use later does not need that database.

### If it fails

The most common problem is a slow or interrupted connection, which looks like this:

```
CondaHTTPError: HTTP 000 CONNECTION FAILED for url <https://conda.anaconda.org/...>
```

Conda keeps what it has already downloaded, so **simply run the same command again**. If it keeps timing out, give Conda more patience and try once more:

```
$ conda config --set remote_read_timeout_secs 120
```

Then repeat the `conda create` command.

If you ever want to start over, remove the environment and repeat Step 2:

```
$ conda deactivate
$ conda env remove -n bioinfo
```

## Step 3: Activate the environment

An environment does nothing until you **activate** it.

```
$ conda activate bioinfo
(bioinfo) yourname@YOUR-PC:~$
```

The `(bioinfo)` at the start of the prompt tells you which environment is active. To switch it off:

```
(bioinfo) $ conda deactivate
(base) $
```

> **You must type `conda activate bioinfo` in every new terminal window.** If a tool says "command not found" and you are sure it is installed, check your prompt: it almost certainly does not say `(bioinfo)`.

## Step 4: Check that every tool works

With `(bioinfo)` showing, ask the terminal where each program lives. `which` prints the location of a program:

```
$ which seqkit fastqc fastp bwa minimap2 samtools bcftools NanoPlot efetch kraken2 ktImportText
/home/yourname/miniforge3/envs/bioinfo/bin/seqkit
/home/yourname/miniforge3/envs/bioinfo/bin/fastqc
/home/yourname/miniforge3/envs/bioinfo/bin/fastp
/home/yourname/miniforge3/envs/bioinfo/bin/bwa
/home/yourname/miniforge3/envs/bioinfo/bin/minimap2
/home/yourname/miniforge3/envs/bioinfo/bin/samtools
/home/yourname/miniforge3/envs/bioinfo/bin/bcftools
/home/yourname/miniforge3/envs/bioinfo/bin/NanoPlot
/home/yourname/miniforge3/envs/bioinfo/bin/efetch
/home/yourname/miniforge3/envs/bioinfo/bin/kraken2
/home/yourname/miniforge3/envs/bioinfo/bin/ktImportText
```

You should see **11 lines**, and every path should contain `envs/bioinfo`. A tool that is missing from the list simply has no line.

Now check that the programs run, and note their versions:

```
$ seqkit version
seqkit v2.14.0
$ fastqc --version
FastQC v0.13.0
$ fastp --version
fastp 1.4.0
$ bwa 2>&1 | grep Version
Version: 0.7.19-r1273
$ minimap2 --version
2.31-r1302
$ samtools --version | head -n 1
samtools 1.25
$ bcftools --version | head -n 1
bcftools 1.25
$ NanoPlot --version
NanoPlot 1.48.0
$ kraken2 --version | head -n 1
Kraken version 2.17.2
```

**Your version numbers may be a little newer than these.** That is fine. The programs change slowly, but if a command behaves differently from the lesson, tell a trainer which version you have.

(`bwa` is run with `2>&1 | grep Version` because it prints its details as an error message when run with no options. `2>&1` joins that message to the normal output, and `grep Version` picks out the line we want.)

### See what is installed

```
$ conda list | head -n 8
```

This lists the packages in the active environment, with their versions and where they came from. Look for `bioconda` or `conda-forge` in the last column.

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `conda: command not found` | Conda is not set up in this terminal | Close and reopen the terminal. If it persists, see the `conda init` fix in Module 0, Part 6 |
| `EnvironmentNameNotFound: Could not find conda environment: bioinfo` | The environment does not exist yet | Follow Path B |
| Prompt starts with `(base)` and tools are missing | The environment is not active | `conda activate bioinfo` |
| `CondaHTTPError` or timeouts during `conda create` | The connection was interrupted | Run the same command again; see "If it fails" above |
| `which` shows fewer than 11 lines | A package did not install | Run the `conda create` command again with `-n bioinfo` replaced by the existing name, or remove and rebuild the environment |
| `No space left on device` | Your disk is full | Free up space (you need about 5 GB) and try again |
| `PackagesNotFoundError` | A package name was mistyped | Copy the command again exactly as written |

## Practice check

1. What does `conda activate bioinfo` do?
2. How can you tell, from the prompt alone, which environment is active?
3. Why do we not install the tools into `base`?
4. `fastqc: command not found`. Your prompt begins with `(base)`. What do you try first?

### Answers

1. It switches on the `bioinfo` environment, so the tools installed in it can be found and run.
2. The name in brackets at the start of the prompt, for example `(bioinfo)`.
3. Keeping one environment per project avoids conflicts between programs and makes it easy to delete and rebuild the environment if something breaks.
4. `conda activate bioinfo`, then try `fastqc --version` again.

## Summary of commands

| Command | What it does |
|---|---|
| `conda --version` | Shows the Conda version |
| `conda env list` | Lists your environments (`*` marks the active one) |
| `conda create -n name -c channel packages` | Creates an environment |
| `conda activate name` | Switches an environment on |
| `conda deactivate` | Switches the current environment off |
| `conda env remove -n name` | Deletes an environment |
| `conda list` | Lists packages in the active environment |
| `which tool` | Shows where a program is |
| `tool --version` | Shows a program's version |

## What comes next

[Lesson 2.3: Files and folders](Lesson_3_Files_and_Folders.md)
