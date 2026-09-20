# ============================================================
# Week 04 - Download and QC FASTQ reads from the SRA
# Organism: Caenorhabditis elegans (the genome chosen in Week 02)
# ============================================================

# ---- Configuration (override on the command line, e.g. make all N=50000) ----
# C. elegans RNA-seq, paired-end Illumina (HiSeq X Ten)
SRR    ?= SRR8240860
# descriptive name used to rename the FASTQ files
SAMPLE ?= celegans_rnaseq
# number of reads (spots) to download - a subset, not the whole run
N      ?= 10000

# Directories named after the data type
READS = reads
TRIM  = trimmed
QC    = qc

R1 = $(READS)/$(SAMPLE)_R1.fastq
R2 = $(READS)/$(SAMPLE)_R2.fastq
T1 = $(TRIM)/$(SAMPLE)_R1.fastq
T2 = $(TRIM)/$(SAMPLE)_R2.fastq

SHELL := bash
.SHELLFLAGS := -eu -o pipefail -c
.DEFAULT_GOAL := help

help:  ## Show the available commands
	@echo "Usage: make <target>   (override SRR=, SAMPLE=, N= as needed)"
	@echo ""
	@echo "  download      Download the first N reads of $(SRR) into $(READS)/"
	@echo "  fastqc        FastQC report on the raw reads"
	@echo "  trim          Trim/filter the reads with fastp into $(TRIM)/"
	@echo "  fastqc-trim   FastQC report on the trimmed reads"
	@echo "  all           Run the whole pipeline"
	@echo "  clean         Remove all generated files"

# ---- Download the first N reads, then rename to descriptive names ----
download: $(R1)  ## Download a subset of reads

$(R1):
	mkdir -p $(READS)
	fastq-dump -X $(N) --split-files -O $(READS) $(SRR)
	mv $(READS)/$(SRR)_1.fastq $(R1)
	mv $(READS)/$(SRR)_2.fastq $(R2)

# ---- QC visualization on the RAW reads ----
fastqc: $(R1)  ## FastQC on raw reads
	mkdir -p $(QC)/raw
	fastqc $(R1) $(R2) -o $(QC)/raw

# ---- Trim/filter with fastp ----
trim: $(T1)  ## Trim reads with fastp

$(T1): $(R1)
	mkdir -p $(TRIM) $(QC)
	fastp -i $(R1) -I $(R2) -o $(T1) -O $(T2) --html $(QC)/fastp.html --json $(QC)/fastp.json

# ---- QC visualization on the TRIMMED reads ----
fastqc-trim: $(T1)  ## FastQC on trimmed reads
	mkdir -p $(QC)/trimmed
	fastqc $(T1) $(T2) -o $(QC)/trimmed

all: download fastqc trim fastqc-trim  ## Run the whole pipeline

clean:  ## Remove all generated files
	rm -rf $(READS) $(TRIM) $(QC)

.PHONY: help download fastqc trim fastqc-trim all clean
