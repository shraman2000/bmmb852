# Week 02 — Obtain and Visualize Genomic Data

**Author:** Shraman Jana
**Organism:** *Caenorhabditis elegans* (nematode)
**Assembly:** GCF_000002985.6 (WBcel235), RefSeq — [NCBI record](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_000002985.6/)

*C. elegans* is the foundational model organism in the biology of aging: the
insulin/IGF-1 signaling pathway and the longevity genes *daf-2* and *daf-16*, as
well as the genetics of dietary-restriction lifespan extension, were first worked
out in this worm. It is not a genome used in the course book or lectures, it is
fully sequenced and richly annotated, and at ~100 Mb across six chromosomes it is
small enough to download and inspect quickly while still being a true eukaryote.

---

## Part 1 — Download the data

### Files

```
week02/
├── Makefile        # downloads + decompresses the data and reports statistics
├── README.md       # this file
└── refs/           # created by `make download`; holds the genome + annotation
    ├── genome.fa   # FASTA sequence
    └── genome.gff  # GFF3 annotation
```

The data is kept in a dedicated `refs/` directory rather than the working
directory, so nothing is dumped loose.

### How to run the Makefile

From inside the `week02` directory, in the `bioinfo` environment:

```bash
micromamba activate bioinfo

make            # shows the list of available commands
make download   # downloads and decompresses the FASTA + GFF into refs/
make stats      # prints genome size, sequence count, and annotation counts
make clean      # deletes refs/ so you can start fresh
```

To use a **different genome**, change only the `ACC` and `URL` lines at the top of
the Makefile — everything else is generic.

---

## Answers to the questions

# Genome size (bp):
100286401
# Number of sequences (chromosomes / replicons):
7
# Sequence names:
>NC_003279.8 Caenorhabditis elegans chromosome I
>NC_003280.10 Caenorhabditis elegans chromosome II
>NC_003281.10 Caenorhabditis elegans chromosome III
>NC_003282.8 Caenorhabditis elegans chromosome IV
>NC_003283.11 Caenorhabditis elegans chromosome V
>NC_003284.9 Caenorhabditis elegans chromosome X
>NC_001328.1 Caenorhabditis elegans mitochondrion, complete genome
# Total annotation lines in GFF (excluding comments):
547615
# Feature types and their counts:
 239332 exon
 204604 CDS
  44795 gene
  30562 mRNA
  15363 piRNA
   7779 ncRNA
   2131 pseudogene
    658 transcript
    634 tRNA
    458 miRNA
    346 snoRNA
    276 primary_transcript
    209 pseudogenic_tRNA
    202 lncRNA
    129 snRNA
    104 antisense_RNA
     22 rRNA
      7 region
      2 sequence_feature
      1 scRNA
      1 pseudogenic_rRNA
# Number of genes:
44795

**How large is the genome? How many chromosomes does it have?**
The genome is about **100 Mb** (~100,286,401 bp). *C. elegans* has **six
chromosomes** — five autosomes (I–V) and one sex chromosome (X). Note the FASTA
contains **seven** sequences, because it also includes the small mitochondrial
genome (MtDNA); that is an organellar genome, not a nuclear chromosome, so the
chromosome count is six.

**How many annotations are in the annotation file?**
The GFF3 file contains 547,615 annotation lines across 21 feature types. The most abundant are exon (239,332), CDS (204,604), and gene (44,795). The gene count (44,795) is much higher than the ~20,000 protein-coding genes because RefSeq annotates many non-coding gene types too — notably 15,363 piRNAs, plus ncRNAs, tRNAs, and pseudogenes.

**How complete is this genomic build, in your opinion?**
In my opinion this is a highly complete, mature genome build. WBcel235 is a chromosome-level reference where each of the six chromosomes is a single continuous sequence with no scaffolding gaps, and it also includes the full mitochondrial genome.The annotation depth i.e., over half a million features covering protein-coding genes, non-coding RNAs, and pseudogenes reflects that maturity rather than a rough first draft. As I am currently involved in aging biology research, I find this especially valuable as so much of what we know about the genetics of lifespan (insulin/IGF-1 signaling, daf-2/daf-16) rests on this worm, and reliable longevity research depends on exactly this kind of trustworthy, richly annotated reference.

---

## Part 2 — Visualize the genome in IGV

Load `refs/genome.fa` as the genome (Genomes → Load Genome from File) and
`refs/genome.gff` as a track (File → Load from File) in IGV.

> The items below must be answered from your **own** IGV session, with
> screenshots. Guidance on how to read each answer is given, but the observations
> and images have to be yours.

**How tightly packed are the genes? Estimate the gene-to-gene distance.**
*C. elegans* is gene-dense for a eukaryote (introns are short and intergenic
regions modest). Zoom to a region a few kb wide, read the start of one gene and
the end of the previous one off the coordinate ruler, and estimate the average
intergenic gap. *(Insert your estimate + screenshot.)*

**Pick a coordinate and inspect the sequence around it.**
Zoom in until IGV shows individual bases, note the coordinate you picked, and
capture the surrounding sequence. *(Insert coordinate + screenshot.)*

**Describe all six reading frames at that coordinate.**
Any position sits in six possible frames: three on the forward strand (starting at
the base, +1, and +2) and three on the reverse-complement strand. Turn on IGV's
3-frame translation (right-click the sequence track) to see the forward frames,
then view the reverse strand for the other three. *(List the codon/frame for each
of the six and include a screenshot.)*

**Identify the type of feature displayed as a data track.**
Describe what the loaded GFF track shows (e.g., gene / mRNA / CDS features).
*(State the feature type + screenshot.)*

**Color features by strand orientation.**
Right-click the annotation track → **Color by strand**, so + and − strand features
are shown in different colors. *(Insert screenshot.)*

*(Add screenshots below — drag images into the `week02` folder, e.g. `img/`, and
reference them like `![gene density](img/density.png)`.)*
