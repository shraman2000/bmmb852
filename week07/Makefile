# ============================================================
# Week 07 - Call variants and generate a VCF file
# Organism: Caenorhabditis elegans (WBcel235, from Week 02/05)
# Reads:    SRR065390 (N2 whole-genome sequencing, Illumina)
# Builds on the Week 05 alignment: genome -> reads -> BAM -> VCF
# ============================================================

# ---- Configuration ----
GACC       = GCF_000002985.6_WBcel235
GENOME_URL = https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/002/985/$(GACC)

SRR    ?= SRR065390
SAMPLE ?= celegans_wgs
# 10x over the 100.3 Mb genome with 2x100 bp reads needs ~5.0M spots
N      ?= 5100000
THREADS ?= 4

# Directories named after the data type
REFS  = refs
READS = reads
BAMD  = bam
VCFD  = vcf

REF = $(REFS)/genome.fa
R1  = $(READS)/$(SAMPLE)_R1.fastq
R2  = $(READS)/$(SAMPLE)_R2.fastq
BAM = $(BAMD)/$(SAMPLE).bam
VCF = $(VCFD)/$(SAMPLE).vcf.gz

SHELL := bash
.SHELLFLAGS := -eu -o pipefail -c
.DEFAULT_GOAL := help

help:  ## Show the available commands
	@echo "Usage: make <target>   (override SRR=, SAMPLE=, N=, THREADS=)"
	@echo ""
	@echo "  genome   Download the reference genome and build the BWA index"
	@echo "  reads    Download the first N spots of $(SRR)"
	@echo "  align    Align reads -> sorted, indexed BAM"
	@echo "  vcf      Call variants -> compressed, indexed VCF"
	@echo "  stats    Report variant statistics (bcftools stats)"
	@echo "  all      Run the whole pipeline (genome->reads->align->vcf->stats)"
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

# ---- Download reads (paired or single auto-handled) ----
reads: $(R1)  ## Download N spots and rename
$(R1):
	mkdir -p $(READS)
	fastq-dump -X $(N) --split-files -O $(READS) $(SRR)
	mv $(READS)/$(SRR)_1.fastq $(R1)
	[ -f $(READS)/$(SRR)_2.fastq ] && mv $(READS)/$(SRR)_2.fastq $(R2) || true

# ---- Align -> sorted, indexed BAM ----
align: $(BAM)  ## Produce a sorted, indexed BAM
$(BAM): $(REF).bwt $(R1)
	mkdir -p $(BAMD)
	if [ -s $(R2) ]; then \
	  bwa mem -t $(THREADS) $(REF) $(R1) $(R2) | samtools sort -@ $(THREADS) -o $(BAM) - ; \
	else \
	  bwa mem -t $(THREADS) $(REF) $(R1) | samtools sort -@ $(THREADS) -o $(BAM) - ; \
	fi
	samtools index $(BAM)

# ---- Call variants -> compressed, indexed VCF ----
vcf: $(VCF)  ## Call variants with bcftools
$(VCF): $(BAM) $(REF)
	mkdir -p $(VCFD)
	bcftools mpileup -Ou -f $(REF) $(BAM) \
	  | bcftools call -mv -Oz -o $(VCF)
	bcftools index $(VCF)

# ---- Variant statistics ----
stats: $(VCF)  ## Report variant statistics
	bcftools stats $(VCF) > $(VCFD)/stats.txt
	@echo "# Total variant records:"
	@bcftools view -H $(VCF) | wc -l
	@echo "# SNPs:"
	@bcftools view -H -v snps $(VCF) | wc -l
	@echo "# Indels:"
	@bcftools view -H -v indels $(VCF) | wc -l
	@echo "# Ts/Tv and summary are in $(VCFD)/stats.txt"

all: genome reads align vcf stats  ## Run the whole pipeline

clean:  ## Remove all generated files
	rm -rf $(REFS) $(READS) $(BAMD) $(VCFD)

.PHONY: help genome reads align vcf stats all clean
