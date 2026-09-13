# Week 03:Code Review and Pull Request

**Author:** Shraman Jana
**Repository reviewed:** [Kny-Le/BMMB852_KL:Week02](https://github.com/Kny-Le/BMMB852_KL/tree/main/Week02)
**Genome in that assignment:** *Helicobacter pylori* (assembly GCF_902846105.1)

For this assignment I reviewed Kenny's Week 2 submission: I forked it,
checked it for safety, tested whether it reproduces, verified that the code does
what Kenny claimed, compared it with my own solution, and contributed a fix
back as a pull request.

---

## 1. Fork and clone

I forked Kenny's repository on GitHub (so the copy lives under my account) and
cloned my fork to my machine:

```bash
git clone https://github.com/shraman2000/BMMB852_KL.git
cd BMMB852_KL/Week02/Makefile
```

## 2. Security check-Is the code doing anything dangerous?

Before running anything I inspected the Makefile (`cat Makefile`). It only:

- downloads the genome from the official NCBI datasets API with `curl`,
- unzips the archive into local `fasta/` and `gff/` folders, and
- runs `samtools`, `bgzip`, and `tabix` to index the outputs.

The `clean` target removes only the local `fasta`, `gff`, and archive. There are
no system-level operations, no piping of remote content into a shell, and no
access to credentials. **The code is safe to run.**

## 3. Reproducibility-Does the code run and produce the stated outputs?

To test reproducibility I deleted the generated files and rebuilt from scratch:

```bash
make clean
make all
ls fasta gff
```

Output (pasted from my terminal):

```text
$ make all
curl --fail --location --silent --show-error --output GCF_902846105.1.zip '<NCBI datasets URL>'
mkdir -p fasta
unzip -p GCF_902846105.1.zip 'ncbi_dataset/data/GCF_902846105.1/*.fna' > fasta/HPylori_NZ_CADHBV000000000.1.fna.tmp
mv fasta/HPylori_NZ_CADHBV000000000.1.fna.tmp fasta/HPylori_NZ_CADHBV000000000.1.fna
mkdir -p gff
unzip -p GCF_902846105.1.zip 'ncbi_dataset/data/GCF_902846105.1/genomic.gff' > gff/HPylori_NZ_CADHBV000000000.1.gff.tmp
mv gff/HPylori_NZ_CADHBV000000000.1.gff.tmp gff/HPylori_NZ_CADHBV000000000.1.gff
samtools faidx fasta/HPylori_NZ_CADHBV000000000.1.fna
bgzip --force --keep gff/HPylori_NZ_CADHBV000000000.1.gff
tabix --force --preset gff gff/HPylori_NZ_CADHBV000000000.1.gff.gz

$ ls fasta gff
fasta:
HPylori_NZ_CADHBV000000000.1.fna  HPylori_NZ_CADHBV000000000.1.fna.fai
gff:
HPylori_NZ_CADHBV000000000.1.gff  HPylori_NZ_CADHBV000000000.1.gff.gz  HPylori_NZ_CADHBV000000000.1.gff.gz.tbi
```

The Makefile ran end to end and regenerated the FASTA, GFF, and all three index
files-matching what the README describes.

## 4. Verifying the author's reported results

I re-ran Kenny's own analysis commands on the freshly generated files to
confirm the numbers in their README are correct.

**Genome size and contig count:**

```bash
awk '/^>/{c++} !/^>/{b+=length($0)} END{print "Contigs:",c; print "Total bp:",b}' \
  fasta/HPylori_NZ_CADHBV000000000.1.fna
```

Output:

```text
Contigs: 49
Total bp: 1657706
```

**Number of annotations in the GFF:**

```bash
zgrep -v '^#' gff/HPylori_NZ_CADHBV000000000.1.gff.gz | wc -l
```

Output:

```text
3347
```

These match Kenny's reported values (49 contigs, ~1.66 Mb, 3,347
annotations), so the code does whatever mentioned.

## 5. Evaluation of the README

The README is clear and well organized. It uses descriptive section headers,
shows the exact command for each step, explains what individual flags do
(e.g. `zgrep`, `-v`, `^#`), pastes the outputs, and includes IGV screenshots. It
also goes beyond the brief by adding a BUSCO completeness analysis. The committed
Makefile is clean and correctly tab-indented and includes an indexing target.
Overall a reader can follow it and reproduce the work.

The main issue I found is a **factual contradiction** in the "How many
chromosomes" answer: the README states the genome "is a singular circular
chromosome that is separated into 49 different contigs." A 49-contig assembly is
a *draft*-it has **not** been closed into a single circular sequence. So
describing it as "a singular circular chromosome" is contradictory. The author's
own BUSCO result (10.5% missing) is consistent with a draft rather than a
finished build.

## 6. Comparison with my own solution (AI-assisted)

The assignment asks the AI agent to compare the two submissions and judge which
is better. I used the AI agent in VS Code, giving it both this submission and my
own Week 2 assignment (*C. elegans*, WBcel235) and asking which is better. Its
assessment, which matches my own reading:

- The genomes are in different completeness classes — Kenny's one is a **draft**
  assembly (49 contigs), mine one is a **finished, chromosome-level** reference
  (6 chromosomes, no gaps).
- Kenny's submission is richer in analytical extras: the BUSCO completeness step
  and the per-flag command explanations are nice which mine does not have.
- Mine is stronger on accuracy for the chromosome question (I distinguished the
  six nuclear chromosomes from the mitochondrial sequence), and it keeps the
  large data out of the repo via `.gitignore`.

**Which is better?** I would treat both in level terms with different strengths
rather than one being simply better-Kenny's one is more exploratory and analytical,
mine one is more accurate and reproducible on the specific questions asked. I also
note that an AI agent can be biased toward whichever codebase is already loaded
in its context, so I weighed its opinion against my own inspection.

## 7. Summary of findings

Kenny's Week 2 submission is a solid, safe, and reproducible piece of work: the
Makefile downloads and indexes the *H. pylori* genome without any dangerous
operations, and rebuilding from scratch regenerated every file and matched the
author's reported numbers (49 contigs, ~1.66 Mb, 3,347 annotations). The README
is readable and well structured, with clear commands, explanations, and
screenshots, and it adds a BUSCO analysis beyond the requirements. The one
substantive problem I found is the contradictory description of the genome as "a singular
circular chromosome that is separated into 49 different contigs"-a 49-contig
build is a draft, not a closed chromosome. This is the issue I addressed in my
pull request.

## 8. Change contributed (pull request)

I fixed the chromosome/contig contradiction in the Week 2 README. I edited the
sentence to describe the assembly accurately as a 49-contig draft (and corrected
the "singluar" typo):

> **Before:** This bacterium has a singluar circular chromosome that is separated
> into 49 different contigs.
>
> **After:** This assembly is a draft genome in 49 contigs rather than a single
> closed sequence. *H. pylori* has one circular chromosome in nature, but this
> build has not been assembled into a single contig-consistent with the BUSCO
> result (10.5% missing) above.

I committed this to my fork and opened a pull request to the original repository.

**Pull request:** [https://github.com/Kny-Le/BMMB852_KL/pull/2]
