# ============================================================
# Week 10 - ChIP-seq / ATAC-seq peak-calling workflow
# Organism: Caenorhabditis elegans (WBcel235, GCF_000002985.6)
# Reuses the Week 09 design-driven batch pipeline, but replaces
# variant calling with MACS2 peak calling (condition vs background).
#
# design.csv columns: sample,group,run,control
#   group   = chip (condition) | input (background)
#   control = the input sample used as background for a chip sample
#             (blank for input rows)
# ============================================================

# ---- Configuration ----
GACC       = GCF_000002985.6_WBcel235
GENOME_URL = https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/002/985/$(GACC)
DESIGN     ?= design.csv
# reads downloaded per sample (first N); keep small enough for a laptop
N          ?= 3000000
# MACS2 effective genome size for C. elegans (~9e7 mappable bp)
GSIZE      ?= 9e7
# Peak caller: bioinfo is pinned to Python 3.12, which MACS2 does not support,
# so MACS2 lives in its own env (created once - see README) and is called via
# micromamba run. Override MACS= if yours is installed differently.
MACS       ?= micromamba run -n macs macs2
THREADS    ?= 4

REFS = refs
READS = reads
BAMD = bam
PEAKS = peaks
REF = $(REFS)/genome.fa

SAMPLE ?=
RUN    ?=

# All samples, and the chip (condition) samples, read from design.csv
SAMPLES    := $(shell tail -n +2 $(DESIGN) | cut -d, -f1)
CHIPS      := $(shell awk -F, 'NR>1 && $$2=="chip"{print $$1}' $(DESIGN))

SHELL := bash
.SHELLFLAGS := -eu -o pipefail -c
.DEFAULT_GOAL := help
.NOTPARALLEL:

help:  ## Show the available commands
	@echo "Usage: make <target>"
	@echo ""
	@echo "  genome      Download the reference genome and build the BWA index"
	@echo "  align       Align every sample in design.csv -> sorted, indexed BAM"
	@echo "  background  Align just the input (background) samples"
	@echo "  peaks       MACS2 peak calling for each chip sample vs its control"
	@echo "  count       Report the number of peaks per chip sample"
	@echo "  all         genome -> align -> peaks -> count"
	@echo "  clean       Remove all generated files"
	@echo ""
	@echo "  Samples: $(SAMPLES)"
	@echo "  Chip samples: $(CHIPS)"

# ---- Reference genome + BWA index ----
genome: $(REF).bwt  ## Download genome, build index
$(REF):
	mkdir -p $(REFS)
	curl -L $(GENOME_URL)/$(GACC)_genomic.fna.gz -o $(REFS)/genome.fa.gz
	gunzip -f $(REFS)/genome.fa.gz
$(REF).bwt: $(REF)
	bwa index $(REF)
	samtools faidx $(REF)

# ---- Align one sample (first N reads; single- or paired-end auto) ----
$(BAMD)/$(SAMPLE).bam: $(REF).bwt
	mkdir -p $(READS) $(BAMD)
	fastq-dump -X $(N) --split-files -O $(READS) $(RUN)
	mv $(READS)/$(RUN)_1.fastq $(READS)/$(SAMPLE)_R1.fastq
	[ -f $(READS)/$(RUN)_2.fastq ] && mv $(READS)/$(RUN)_2.fastq $(READS)/$(SAMPLE)_R2.fastq || true
	if [ -s $(READS)/$(SAMPLE)_R2.fastq ]; then \
	  bwa mem -T 20 -t $(THREADS) $(REF) $(READS)/$(SAMPLE)_R1.fastq $(READS)/$(SAMPLE)_R2.fastq | samtools sort -@ $(THREADS) -o $(BAMD)/$(SAMPLE).bam - ; \
	else \
	  bwa mem -T 20 -t $(THREADS) $(REF) $(READS)/$(SAMPLE)_R1.fastq | samtools sort -@ $(THREADS) -o $(BAMD)/$(SAMPLE).bam - ; \
	fi
	samtools index $(BAMD)/$(SAMPLE).bam

one: $(BAMD)/$(SAMPLE).bam  ## Align one sample (SAMPLE= RUN=)

# ---- Align every sample in the design file ----
align: $(REF).bwt  ## Align all samples
	@tail -n +2 $(DESIGN) | while IFS=, read sample group run control; do \
	  echo "=== align $$sample ($$group) ==="; \
	  $(MAKE) one SAMPLE=$$sample RUN=$$run; \
	done

# ---- Align only the background (input) samples ----
background: $(REF).bwt  ## Align background/input samples
	@awk -F, 'NR>1 && $$2=="input"' $(DESIGN) | while IFS=, read sample group run control; do \
	  echo "=== background $$sample ==="; \
	  $(MAKE) one SAMPLE=$$sample RUN=$$run; \
	done

# ---- Peak calling: each chip sample vs its control (input) ----
peaks: align  ## MACS2 peaks for each chip sample
	mkdir -p $(PEAKS)
	@awk -F, 'NR>1 && $$2=="chip"' $(DESIGN) | while IFS=, read sample group run control; do \
	  echo "=== peaks $$sample vs $$control ==="; \
	  $(MACS) callpeak -t $(BAMD)/$$sample.bam -c $(BAMD)/$$control.bam \
	    -f BAM -g $(GSIZE) -n $$sample --outdir $(PEAKS) 2> $(PEAKS)/$$sample.log; \
	done

# ---- Count peaks per chip sample ----
count: ## Report peaks per chip sample
	@echo "# Peaks per sample (narrowPeak line counts):"
	@for s in $(CHIPS); do \
	  n=$$(wc -l < $(PEAKS)/$$s'_peaks.narrowPeak' 2>/dev/null || echo 0); \
	  echo "$$s	$$n"; \
	done

all: genome align peaks count  ## Full pipeline

clean:  ## Remove all generated files
	rm -rf $(REFS) $(READS) $(BAMD) $(PEAKS) SRR*/ *.sra

.PHONY: help genome one align background peaks count all clean
