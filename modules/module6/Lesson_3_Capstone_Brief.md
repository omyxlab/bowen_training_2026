# Lesson 6.3: Capstone Brief

**Module 6: Wrap-Up and Next Steps**

## What you will need

- Nothing new to read this lesson
- To do the capstone: your `bioinfo` environment and your Ebola project folder (or the time to rebuild it)

## What you will learn

By the end of this lesson you will be able to:

- Explain what the capstone asks you to do and why
- Find the full instructions
- Plan your time, and know how to submit your work
- Know where to get help

## What the capstone is

The capstone is a take-home project that you do **after** the training, at your own pace, over **one to two weeks**. Its purpose is to make you run an analysis **yourself**, without a trainer typing alongside you. That is where the learning settles.

## The idea

In Module 4 you compared the Ebola sample with a reference isolated in **1976** and found 559 differences. You explained this by saying the reference is a distant relative of the sample. That was a reasonable explanation, but nobody tested it.

The capstone tests it:

> **Use a reference genome that is closer to the sample. What changes, what stays the same, and why?**

You will:

1. Download a **2014 Sierra Leone** Ebola genome as a new reference
2. Map your trimmed reads to it, and measure the mapping and coverage
3. Call the variants, and build the consensus
4. Run a **negative control** against an unrelated virus
5. Compare everything with your Module 3 and 4 results
6. Write up what you found, in plain language, in a file called `findings.txt`

## Why this one?

- **It uses only what you already know.** No new commands, only the ones from Modules 3 and 4, applied to new input.
- **It teaches a lesson that every bioinformatician needs.** The choice of reference shapes your results.
- **It teaches you to test your own explanation**, with a control, instead of assuming it.
- **It is short to run.** The computing takes seconds. Your time goes into thinking and writing.

## The instructions

The full instructions are in the [capstone folder](../../capstone/README.md) of this repository. They are also posted on the Omyxlab website. They include:

- The tasks, step by step, with hints
- The questions for `findings.txt`
- How to pack your work with `tar` and submit it
- A checklist to run through before you send it

## Planning your work

Take it in this order:

1. Rebuild your project if you need to (Lesson 6.1 lists every command).
2. Part 1: the closer reference.
3. Part 2: the negative control.
4. Part 3: write up `findings.txt`.
5. Pack your work and submit it.

Do not rush. If you get stuck, move on to the next part and come back to it.

## Tips

- **Work in one folder, and name your files clearly.** `makona.sorted.bam` and `makona.vcf` tell you which reference they belong to.
- **Change only one thing:** the reference. The reads, the trimming and the filter settings must stay the same, or you cannot tell what caused any difference.
- **Save as you go.** Use `>` to put each result in a file, so you do not have to redo anything.
- **Look back at the lessons.** Each lesson has a "If something looks different" section. Most problems you will meet are in there.
- **Write it up while it is fresh.** Answer each `findings.txt` question straight after you get the number.
- **Explain, do not just report.** "The variants dropped from 559 to N" is a number. "...because..." is the understanding.

## Getting help

- Re-read the relevant lesson and its troubleshooting table.
- Email **bioinformaticsfridays@gmail.com** with a description of the problem, which step you were on, and a screenshot of any error message.
- If you were using the JupyterHub, ask the trainers whether access can be extended, or how to set up your own computer. The [Getting Started guide](../module0/Getting_Started.md) covers installing everything on your own machine.

## Practice check

1. What is the one thing you change in the capstone, and why only one?
2. What is a negative control, and what does it do?
3. What do you submit, and what should you leave out?

### Answers

1. The reference genome. If you change anything else as well, you cannot tell which change caused a difference in the results.
2. It is a run where you expect nothing to happen, here mapping the Ebola reads to an unrelated virus. It checks that your method does not produce results by accident.
3. One compressed file with `findings.txt`, the saved `flagstat` and `coverage` text files, the VCF and consensus files, and your commands history. Leave out the BAM and FASTQ files, which are large and not needed.

## What comes next

[Lesson 6.4: Where next, Bioinformatic Fridays](Lesson_4_Where_Next.md)
