# ============================================================
# Week 05 - Align reads to a genome and generate a BAM file
# Organism: Caenorhabditis elegans (WBcel235, from Week 02)
# Reads:    SRR065390 (N2 whole-genome sequencing, Illumina)
# ============================================================

# ---- Configuration (override on the command line if needed) ----
# Genome assembly (RefSeq)
GACC       = GCF_000002985.6_WBcel235
GENOME_URL = https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/002/985/$(GACC)

# Sequencing run to align
SRR    ?= SRR065390
SAMPLE ?= celegans_wgs
# Number of spots (read pairs for paired data) to download.
# 10x over the 100.3 Mb genome with 2x100 bp reads needs ~5.0M spots.
N      ?= 5100000
THREADS ?= 4

# Directories named after the data type
REFS  = refs
READS = reads
BAMD  = bam

REF = $(REFS)/genome.fa
R1  = $(READS)/$(SAMPLE)_R1.fastq
R2  = $(READS)/$(SAMPLE)_R2.fastq
BAM = $(BAMD)/$(SAMPLE).bam

SHELL := bash
.SHELLFLAGS := -eu -o pipefail -c
.DEFAULT_GOAL := help

help:  ## Show the available commands
	@echo "Usage: make <target>   (override SRR=, SAMPLE=, N=, THREADS= as needed)"
	@echo ""
	@echo "  genome   Download the reference genome and build the BWA index"
	@echo "  reads    Download the first N spots of $(SRR)"
	@echo "  align    Align the reads and produce a sorted, indexed BAM"
	@echo "  stats    Report alignment statistics and coverage"
	@echo "  all      Run the whole pipeline (genome -> reads -> align -> stats)"
	@echo "  clean    Remove all generated files"

# ---- Reference genome + BWA index ----
genome: $(REF).bwt  ## Download genome and build index

$(REF):
	mkdir -p $(REFS)
	curl -L $(GENOME_URL)/$(GACC)_genomic.fna.gz -o $(REFS)/genome.fa.gz
	gunzip -f $(REFS)/genome.fa.gz

$(REF).bwt: $(REF)
	bwa index $(REF)
	samtools faidx $(REF)

# ---- Download a subset of reads (auto-handles paired or single end) ----
reads: $(R1)  ## Download N spots and rename

$(R1):
	mkdir -p $(READS)
	fastq-dump -X $(N) --split-files -O $(READS) $(SRR)
	mv $(READS)/$(SRR)_1.fastq $(R1)
	[ -f $(READS)/$(SRR)_2.fastq ] && mv $(READS)/$(SRR)_2.fastq $(R2) || true

# ---- Align -> sorted, indexed BAM (auto-detects layout) ----
align: $(BAM)  ## Produce a sorted, indexed BAM

$(BAM): $(REF).bwt $(R1)
	mkdir -p $(BAMD)
	if [ -s $(R2) ]; then \
	  bwa mem -t $(THREADS) $(REF) $(R1) $(R2) | samtools sort -@ $(THREADS) -o $(BAM) - ; \
	else \
	  bwa mem -t $(THREADS) $(REF) $(R1) | samtools sort -@ $(THREADS) -o $(BAM) - ; \
	fi
	samtools index $(BAM)

# ---- Statistics report ----
stats: $(BAM)  ## Alignment stats + coverage
	samtools flagstat $(BAM) | tee $(BAMD)/flagstat.txt
	samtools coverage $(BAM) | tee $(BAMD)/coverage.txt
	samtools stats $(BAM) > $(BAMD)/samstats.txt
	@echo "Mean depth across genome:"
	samtools depth -a $(BAM) | awk '{s+=$$3; n++} END{if(n>0) printf "%.2fx over %d positions\n", s/n, n}'

all: genome reads align stats  ## Run the whole pipeline

clean:  ## Remove all generated files
	rm -rf $(REFS) $(READS) $(BAMD)

.PHONY: help genome reads align stats all clean
