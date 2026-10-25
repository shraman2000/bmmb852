# ============================================================
# Week 09 - Design-file-driven multi-sample variant workflow
# Organism: Caenorhabditis elegans (WBcel235, GCF_000002985.6)
# Reads:    SRR065390 (N2 whole-genome sequencing, Illumina)
# design.csv lists the samples; 'make batch' runs the workflow on each,
# then 'make merge' combines all calls into one multi-sample VCF.
# ============================================================

# ---- Configuration ----
GACC       = GCF_000002985.6_WBcel235
GENOME_URL = https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/002/985/$(GACC)
SRR        ?= SRR065390
DESIGN     ?= design.csv
THREADS    ?= 4

REFS = refs
READS = reads
BAMD = bam
VCFD = vcf
REF = $(REFS)/genome.fa

# Per-sample variables (set by the 'batch' loop when it calls 'make one')
SAMPLE ?=
RUN    ?=
START  ?=
END    ?=

# List of samples / expected per-sample VCFs, read from design.csv
SAMPLES := $(shell tail -n +2 $(DESIGN) | cut -d, -f1)
VCFS    := $(addprefix $(VCFD)/,$(addsuffix .vcf.gz,$(SAMPLES)))

SHELL := bash
.SHELLFLAGS := -eu -o pipefail -c
.DEFAULT_GOAL := help
.NOTPARALLEL:

help:  ## Show the available commands
	@echo "Usage: make <target>"
	@echo ""
	@echo "  genome   Download the reference genome and build the BWA index"
	@echo "  sra      Prefetch the SRA run once (fast range extraction after)"
	@echo "  one      Run one sample (needs SAMPLE= RUN= START= END=)"
	@echo "  batch    Read design.csv and run the workflow for every sample"
	@echo "  merge    Merge all per-sample VCFs into one multi-sample VCF"
	@echo "  all      genome -> sra -> batch -> merge"
	@echo "  clean    Remove all generated files"
	@echo ""
	@echo "  Samples from $(DESIGN): $(SAMPLES)"

# ---- Reference genome + BWA index ----
genome: $(REF).bwt  ## Download genome, build index
$(REF):
	mkdir -p $(REFS)
	curl -L $(GENOME_URL)/$(GACC)_genomic.fna.gz -o $(REFS)/genome.fa.gz
	gunzip -f $(REFS)/genome.fa.gz
$(REF).bwt: $(REF)
	bwa index $(REF)
	samtools faidx $(REF)

# ---- Prefetch the SRA run once so range extraction is fast ----
sra:  ## Prefetch the SRA run
	prefetch -f no $(SRR) || true

# ---- Run a single sample (download a spot range -> align -> call) ----
one: $(VCFD)/$(SAMPLE).vcf.gz  ## One sample (SAMPLE= RUN= START= END=)

$(VCFD)/$(SAMPLE).vcf.gz: $(REF).bwt
	mkdir -p $(READS) $(BAMD) $(VCFD)
	fastq-dump -N $(START) -X $(END) --split-files -O $(READS) $(RUN)
	mv $(READS)/$(RUN)_1.fastq $(READS)/$(SAMPLE)_R1.fastq
	[ -f $(READS)/$(RUN)_2.fastq ] && mv $(READS)/$(RUN)_2.fastq $(READS)/$(SAMPLE)_R2.fastq || true
	if [ -s $(READS)/$(SAMPLE)_R2.fastq ]; then \
	  bwa mem -t $(THREADS) -R '@RG\tID:$(SAMPLE)\tSM:$(SAMPLE)\tPL:ILLUMINA' $(REF) $(READS)/$(SAMPLE)_R1.fastq $(READS)/$(SAMPLE)_R2.fastq | samtools sort -@ $(THREADS) -o $(BAMD)/$(SAMPLE).bam - ; \
	else \
	  bwa mem -t $(THREADS) -R '@RG\tID:$(SAMPLE)\tSM:$(SAMPLE)\tPL:ILLUMINA' $(REF) $(READS)/$(SAMPLE)_R1.fastq | samtools sort -@ $(THREADS) -o $(BAMD)/$(SAMPLE).bam - ; \
	fi
	samtools index $(BAMD)/$(SAMPLE).bam
	bcftools mpileup -Ou -f $(REF) $(BAMD)/$(SAMPLE).bam | bcftools call -mv -Oz -o $(VCFD)/$(SAMPLE).vcf.gz
	bcftools index $(VCFD)/$(SAMPLE).vcf.gz

# ---- Read design.csv and run the workflow for each sample ----
batch: $(REF).bwt  ## Run every sample in design.csv
	@tail -n +2 $(DESIGN) | while IFS=, read sample run start end; do \
	  echo "=== sample $$sample ($$run spots $$start-$$end) ==="; \
	  $(MAKE) one SAMPLE=$$sample RUN=$$run START=$$start END=$$end; \
	done

# ---- Merge all per-sample VCFs into one multi-sample VCF ----
merge: ## Merge per-sample VCFs -> multi-sample VCF
	mkdir -p $(VCFD)
	bcftools merge -Oz -o $(VCFD)/merged.vcf.gz $(VCFS)
	bcftools index $(VCFD)/merged.vcf.gz
	@echo "# Samples in merged VCF:"; bcftools query -l $(VCFD)/merged.vcf.gz
	@echo "# Total variant records:"; bcftools view -H $(VCFD)/merged.vcf.gz | wc -l

all: genome sra batch merge  ## Full pipeline

clean:  ## Remove all generated files
	rm -rf $(REFS) $(READS) $(BAMD) $(VCFD) SRR*/ *.sra

.PHONY: help genome sra one batch merge all clean
