# ============================================================
# Week 08 - Multi-sample VCF + variant effect prediction
# Organism: Caenorhabditis elegans (WBcel235, GCF_000002985.6)
# Three samples = three independent read subsets of SRR065390
# Pipeline: genome -> 3x(reads->align->call) -> merge -> annotate
# ============================================================

# ---- Configuration ----
GACC       = GCF_000002985.6_WBcel235
GENOME_URL = https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/002/985/$(GACC)
SRR    ?= SRR065390
# spots (read pairs) per subset; 3 subsets are taken back-to-back
N      ?= 2000000
THREADS ?= 4

REFS = refs
READS = reads
BAMD = bam
VCFD = vcf
REF = $(REFS)/genome.fa

# snpEff database is discovered at run time (name varies by snpEff version)

SHELL := bash
.SHELLFLAGS := -eu -o pipefail -c
.DEFAULT_GOAL := help

help:  ## Show the available commands
	@echo "Usage: make <target>   (override SRR=, N=, THREADS=)"
	@echo ""
	@echo "  genome    Download the reference genome and build the BWA index"
	@echo "  sra       Prefetch the SRA run once (fast range extraction after)"
	@echo "  samples   Build 3 subsets -> align -> per-sample VCF (s1,s2,s3)"
	@echo "  merge     Merge the 3 per-sample VCFs into a multi-sample VCF"
	@echo "  annotate  snpEff variant-effect annotation (renames chroms first)"
	@echo "  coding    List coding/missense variants with gene + protein change"
	@echo "  stats     bcftools stats on the multi-sample VCF"
	@echo "  all       Run the whole pipeline"
	@echo "  listdb    Show the snpEff C. elegans database name"
	@echo "  clean     Remove all generated files"

# ---- Reference genome + BWA index ----
genome: $(REF).bwt  ## Download genome, build index
$(REF):
	mkdir -p $(REFS)
	curl -L $(GENOME_URL)/$(GACC)_genomic.fna.gz -o $(REFS)/genome.fa.gz
	gunzip -f $(REFS)/genome.fa.gz
$(REF).bwt: $(REF)
	bwa index $(REF)
	samtools faidx $(REF)

# ---- Prefetch SRA once so range extraction is fast ----
sra:  ## Prefetch the SRA run
	prefetch -f no $(SRR) || true

# ---- Build 3 subsets, align each (distinct read-group SM), call per-sample VCF ----
samples: $(VCFD)/celegans_s1.vcf.gz $(VCFD)/celegans_s2.vcf.gz $(VCFD)/celegans_s3.vcf.gz  ## 3 samples

# Static pattern would not carry per-sample spot ranges, so the three are explicit.
define SAMPLE_RULE
$(VCFD)/celegans_s$(1).vcf.gz: $(REF).bwt
	mkdir -p $(READS) $(BAMD) $(VCFD)
	fastq-dump -N $(2) -X $(3) --split-files -O $(READS) $(SRR)
	mv $(READS)/$(SRR)_1.fastq $(READS)/celegans_s$(1)_R1.fastq
	[ -f $(READS)/$(SRR)_2.fastq ] && mv $(READS)/$(SRR)_2.fastq $(READS)/celegans_s$(1)_R2.fastq || true
	if [ -s $(READS)/celegans_s$(1)_R2.fastq ]; then \
	  bwa mem -t $(THREADS) -R '@RG\tID:s$(1)\tSM:celegans_s$(1)\tPL:ILLUMINA' $(REF) $(READS)/celegans_s$(1)_R1.fastq $(READS)/celegans_s$(1)_R2.fastq | samtools sort -@ $(THREADS) -o $(BAMD)/celegans_s$(1).bam - ; \
	else \
	  bwa mem -t $(THREADS) -R '@RG\tID:s$(1)\tSM:celegans_s$(1)\tPL:ILLUMINA' $(REF) $(READS)/celegans_s$(1)_R1.fastq | samtools sort -@ $(THREADS) -o $(BAMD)/celegans_s$(1).bam - ; \
	fi
	samtools index $(BAMD)/celegans_s$(1).bam
	bcftools mpileup -Ou -f $(REF) $(BAMD)/celegans_s$(1).bam | bcftools call -mv -Oz -o $(VCFD)/celegans_s$(1).vcf.gz
	bcftools index $(VCFD)/celegans_s$(1).vcf.gz
endef

# spot ranges: s1=1..N, s2=N+1..2N, s3=2N+1..3N
$(eval $(call SAMPLE_RULE,1,1,$(N)))
$(eval $(call SAMPLE_RULE,2,$(shell echo $$(($(N)+1))),$(shell echo $$((2*$(N))))))
$(eval $(call SAMPLE_RULE,3,$(shell echo $$((2*$(N)+1))),$(shell echo $$((3*$(N))))))

# ---- Merge into a multi-sample VCF ----
merge: $(VCFD)/merged.vcf.gz  ## Multi-sample VCF
$(VCFD)/merged.vcf.gz: $(VCFD)/celegans_s1.vcf.gz $(VCFD)/celegans_s2.vcf.gz $(VCFD)/celegans_s3.vcf.gz
	bcftools merge -Oz -o $(VCFD)/merged.vcf.gz $^
	bcftools index $(VCFD)/merged.vcf.gz

# ---- Annotate effects with snpEff (rename RefSeq->Ensembl chrom names first) ----
annotate: $(VCFD)/annotated.vcf  ## snpEff annotation
$(VCFD)/annotated.vcf: $(VCFD)/merged.vcf.gz
	printf 'NC_003279.8\tI\nNC_003280.10\tII\nNC_003281.10\tIII\nNC_003282.8\tIV\nNC_003283.11\tV\nNC_003284.9\tX\nNC_001328.1\tMtDNA\n' > $(VCFD)/chrom_map.txt
	bcftools annotate --rename-chrs $(VCFD)/chrom_map.txt $(VCFD)/merged.vcf.gz -Oz -o $(VCFD)/merged.ens.vcf.gz
	DB=$$(snpEff databases | grep -i 'WBcel235' | tail -1 | cut -f1); \
	echo "Using snpEff database: $$DB"; \
	snpEff -v $$DB $(VCFD)/merged.ens.vcf.gz > $(VCFD)/annotated.vcf

# ---- List coding variants (gene + protein change) ----
coding: $(VCFD)/annotated.vcf  ## Coding/missense variants
	@echo "# CHROM POS REF ALT  EFFECT|GENE|HGVS.p"
	@grep -v '^#' $(VCFD)/annotated.vcf | grep -E 'missense_variant|stop_gained|stop_lost|synonymous_variant' \
	  | awk -F'\t' '{split($$8,a,"ANN="); split(a[2],b,","); split(b[1],c,"|"); print $$1,$$2,$$4,$$5," "c[2]"|"c[4]"|"c[11]}' | head -30

# ---- Statistics ----
stats: $(VCFD)/merged.vcf.gz  ## Variant statistics
	bcftools stats -s - $(VCFD)/merged.vcf.gz > $(VCFD)/stats.txt
	@echo "# Samples in merged VCF:"
	@bcftools query -l $(VCFD)/merged.vcf.gz
	@echo "# Total records / SNPs / indels:"
	@bcftools view -H $(VCFD)/merged.vcf.gz | wc -l
	@bcftools view -H -v snps $(VCFD)/merged.vcf.gz | wc -l
	@bcftools view -H -v indels $(VCFD)/merged.vcf.gz | wc -l

listdb:  ## Show snpEff C. elegans database name
	@snpEff databases | grep -i WBcel235 || echo 'snpEff or DB not found - install: micromamba install -n bioinfo -c bioconda snpeff'

all: genome sra samples merge annotate stats  ## Run the whole pipeline

clean:  ## Remove generated files
	rm -rf $(REFS) $(READS) $(BAMD) $(VCFD) snpEff_genes.txt snpEff_summary.html

.PHONY: help genome sra samples merge annotate coding stats listdb all clean
