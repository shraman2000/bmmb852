# ============================================================
# Week 02 - Obtain genomic data
# Organism : Caenorhabditis elegans (nematode; model organism in aging biology)
# Assembly : GCF_000002985.6 (WBcel235), RefSeq / NCBI
# ============================================================

# ---- Configuration (change these to use a different genome) ----
ACC = GCF_000002985.6_WBcel235
URL = https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/002/985/$(ACC)

# Keep everything in a dedicated folder (do not dump in current dir)
REFS = refs

FASTA_GZ = $(REFS)/$(ACC)_genomic.fna.gz
GFF_GZ   = $(REFS)/$(ACC)_genomic.gff.gz
FASTA    = $(REFS)/genome.fa
GFF      = $(REFS)/genome.gff

# Stop on the first error in a recipe pipeline
SHELL := bash
.SHELLFLAGS := -eu -o pipefail -c

.DEFAULT_GOAL := help

help:  ## Show the available commands
	@echo "Usage: make <target>"
	@echo ""
	@echo "  download   Download + decompress the FASTA and GFF into $(REFS)/"
	@echo "  stats      Print genome size, sequence count, and annotation counts"
	@echo "  clean      Remove all downloaded files"

$(REFS):
	mkdir -p $(REFS)

download: $(FASTA) $(GFF)  ## Download and decompress genome + annotation

$(FASTA): | $(REFS)
	curl -L $(URL)/$(ACC)_genomic.fna.gz -o $(FASTA_GZ)
	gunzip -k -f $(FASTA_GZ)
	mv $(REFS)/$(ACC)_genomic.fna $(FASTA)

$(GFF): | $(REFS)
	curl -L $(URL)/$(ACC)_genomic.gff.gz -o $(GFF_GZ)
	gunzip -k -f $(GFF_GZ)
	mv $(REFS)/$(ACC)_genomic.gff $(GFF)

stats: download  ## Report basic genome statistics
	@echo "# Genome size (bp):"
	@grep -v '^>' $(FASTA) | tr -d '\n' | wc -c
	@echo "# Number of sequences (chromosomes / replicons):"
	@grep -c '^>' $(FASTA)
	@echo "# Sequence names:"
	@grep '^>' $(FASTA)
	@echo "# Total annotation lines in GFF (excluding comments):"
	@grep -vc '^#' $(GFF)
	@echo "# Feature types and their counts:"
	@grep -v '^#' $(GFF) | cut -f3 | sort | uniq -c | sort -rn
	@echo "# Number of genes:"
	@grep -v '^#' $(GFF) | awk -F'\t' '$$3=="gene"' | wc -l

clean:  ## Remove the refs directory
	rm -rf $(REFS)

.PHONY: help download stats clean
