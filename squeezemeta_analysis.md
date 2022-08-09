# Squeezemeta analysis

This document outlines the workflow required to perform Squeezemeta (SQM) on metagenome sequences for the Australian Microbiome database.

The workflow consists of the following stages:

	A] Sequence analysis
	  1. Megahit assembly 
	  2. Squeezemeta analysis
	B] Post analysis processing
	  1. Generate a sorted bam file
	  2. File organisation and renaming

### Software used

The following software is used:

	1. Megahit version 1.2.9 (Li et al., 2015; Li et al., 2016)
	2. Squeezemeta version 1.5.1 (Tamames and Puente-Sanchez 2019)
	3. Squeezemeta database build date for this workflow is: Jul 10 16:26:35 2022
	4. Samtools version 1.12 (Danecek, et al., 2021)
	5. Perl 5.32.1
	6. Python 3

### A] Sequence analysis
#### A-1] Megahit assembly

Data is processed on a per sample basis. The QC sequence files are assembled using Megahit. An example of the structure of the command used is:

	megahit --min-contig-len 500 -1 sampleID_*R1p.fastq.gz -2 sampleID_*R2p.fastq.gz -r sampleID_*merged.fastq.gz -r sampleID_*R1R2u.fastq.gz -m 0.9 -t 24 -o ./${plate}.megahit_asm

#### A-2] Squeezemeta analysis

Squeezemeta is performed using the output of Megahit as an external assembly. 

The SQM `samples` file points to all original (non-QC) files for the sample being analysed, which are used to map against the external Megahit assembly. An example of a SQM tab separated `samples` file with a sample sequenced on 2 lanes of a flow cell would be generated with the following information:

	sampleID_SQM	sampleID-flowcellID-laneX_*_R1.fastq.gz 	pair1
	sampleID_SQM	sampleID-flowcellID-laneX_*_R2.fastq.gz 	pair2
	sampleID_SQM	sampleID-flowcellID-laneY_*_R1.fastq.gz 	pair1
	sampleID_SQM	sampleID-flowcellID-laneY_*_R2.fastq.gz 	pair2

SQM is run on the sample with using a command with the structure:

	perl <path_to_squeezemeta>/SqueezeMeta/scripts/SqueezeMeta.pl -m sequential -s sampleID.samples -f <path_to_non-QC_fastq.gz> -extassembly sampleID_final.contigs.fa -t 24

### B] Post analysis processing
#### B-1] Generate a sorted bam file

A sorted bam file is produced in a 2-step process from the sam file produced by SQM using commands with the following structures:

	1. samtools view -S -b sampleID.sam > sampleID_sqm.bam

	2. samtools sort sampleID_sqm.bam -o sampleID_MGSD_CSIRO.sqm.sorted.bam

#### B-2] File organisation and renaming

Metagenomic bins and associated taxonomy files identified in the analysis are packaged into a zip file. The zip package contains an additional file containing md5 checksums for each file in the packet. Naming convention for the zip packet is as below:

	sampleID_MGSD_CSIRO.sqm.bins.zip

Following analysis, log files relating to the SQM workflow are appended with the AM sampleID and packaged into a zip file. Information relating to the build date of the Squeezemeta database used in the analysis is added to the methods file by concatenating the `DB_BUILD_DATE` file held in the Squeezemeta database directory. The zip package contains an additional file containing md5 checksums for each file in the packet, with the package being named in the format:

	sampleID_MGSD_CSIRO.sqm.logs.zip

All remaining output files are appended with a `prefix` defining the AM sampleID; the file is metagenomic secondary data; the institution performing the analysis; and the analysis workflow prior to ingest into the AM data portal. The `prefix` has the format:

	sampleID_MGSD_CSIRO.sqm.

### References

D. Li, C.M. Liu, R. Luo, K. Sdakane, and T.W. Lam (2015) MEGAHIT: An ultra-fast single-node solution for large and complex metagenomics assembly via succinct de Bruijn graph. Bioinformatics, doi: 10.1093/bioinformatics/btv033 [PMID:25609793].

D. Li, R. Luo, C.M. Liu, C.M. Leung, H.F. Ting, K. Sadakane, H. Yamashita, and T.W. Lam (2016). MEGAHIT v1.0: A Fast and Scalable Metagenome Assembler driven by Advanced Methodologies and Community Practices. Methods 102:3-11.

J. Tamames and F. Puente-Sanchez (2019). SqueezeMeta, A Highly Portable, Fully Automatic Metagenomic Analysis Pipeline. Frontiers in Microbiology 9:3349. doi: 10.3389/fmicb.2018.03349.

P. Danecek, J.K. Bonfield, J. Liddle, J. Marshall, V. Ohan, M.O. Pollard, A. Whitwham, T. Keane, S.A. McCarthy, R.M. Davies and H. Li (2021). Twelve years of SAMtools and BCFtools. GigaScience, Volume 10: giab008 https://doi.org/10.1093/gigascience/giab008.
