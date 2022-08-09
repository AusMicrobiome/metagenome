# Metagenome Sequence Quality Control

This document outlines the workflow required to perform quality control (QC) on metagenome sequences for the Australian Microbiome database.

The workflow consists of the following stages:

	A] File preparation and naming
	  1. Paired end R1 and R2 fastq.gz files are renamed with a `prefix` containing the sampleID, flowID and lane number
	B] Sequence screening
	  1. Removal of adapters, polyA and phiX from paired end files
	C] Sequence merging
	  1. Merge sequences
	  2. Identify paired and unpaired sequences
	D] Post QC processing
	  1. Rename QC samples and concatenate samples that have been sequenced more than once  
	  2. Produce summary statistics of QC files

### Software used
The following software is used:

	1. BBMap version 38.90 (Bushnell, 2014)
	2. FastQC version 0.11.9 (Andrews, 2010).
	3. MultiQC version1.11 (Ewels et al., 2016)
	4. Python3

### A] File preparation and naming
#### A-1] File naming

Data is processed on a per sample basis as separate paired `R1` and `R2` `fastq.gz` files. Each file is renamed with a `prefix` consisting of the AM sample ID, illumina flowcell ID, and lane number in the format:

	sampleID-flowcellID-lane_

### B] Sequence Screening
#### B-1] Removal of adapters, polyA and phiX from paired end files
Screening of sequences for adapters, polyA, and phiX is performed with BBduk using the default parameters with the trimpolyA parameter set at 7. An example of the structure of the command used is:

	bbduk.sh in=prefix*R1*.fastq.gz in2=prefix*R2*.fastq.gz out=prefix*_bbduk_Trim_R1*.fastq.gz out2=prefix*_bbduk_Trim_R2*.fastq.gz ref=adapters,phix trimpolya=7 stats=prefix*_bbduk_Trim_R1*_stats.txt refstats=prefix*_bbduk_Trim_R1*_refstats.txt tbo=t overwrite=t

### C] Sequence Merging
#### C-1] Merge sequences
Sequence merging is performed using BBmerge under default conditions. An example of the structure of the command used is:

	bbmerge.sh in1=prefix*_bbduk_Trim_R1*.fastq.gz in2=prefix*_bbduk_Trim_R2*.fastq.gz out=prefix*_merged*.fastq.gz outu1=prefix*_bbmerge_R1*.fastq.gz outu2=prefix*_bbmerge_R2*.fastq.gz overwrite=t

#### C-2] Identify paired and unpaired sequences
Following merging, unmerged `R1` and `R2` sequences are quality trimmed. Any unpaired sequences following quality trimming are identified and saved. An example of the structure of the command used is:

	bbduk.sh in1=prefix*_bbmerge_R1*.fastq.gz in2=prefix*_bbmerge_R2*.fastq.gz out1=prefix*_R1p*.fastq.gz out2=prefix*_R2p*.fastq.gz outs=prefix*_R1R2u*.fastq.gz qtrim=r trimq=10 overwrite=t

### D] Post QC processing
#### D-1] Rename QC samples
Following sequence QC, files are renamed. If the sample ID has not been previously sequenced, filles are renamed to the following format:

	sampleID_flowID-laneID_R1p.fastq.gz
	sampleID_flowID-laneID_R2p.fastq.gz
	sampleID_flowID-laneID_merged.fastq.gz
	sampleID_flowID-laneID_R1R2u.fastq.gz

If the sample ID has been previously sequenced (e.g., on a different plate, or on multiple lanes) files are concatenated and renamed to the following format:

	sampleID_combined_R1p.fastq.gz
	sampleID_combined_R2p.fastq.gz
	sampleID_combined_merged.fastq.gz
	sampleID_combined_R1R2u.fastq.gz

QC metagenomic reads files are provided to the Australian microbiome web portal on a per sample basis in a zip package. Each zip package contains an additional file containing md5 checksums for each QC metagenome file. Naming convention for the zip packet is as below:

	sampleID_MGSD_CSIRO.qc.fastq.zip

#### D-2] Produce summary statistics of QC files

Summary information showing the number of input reads, and information relating to adapter and phiX removal are produced from the BBtools refstats file. If multiple R1 and R2 files were processed for the sampleID, files are concatenated.

Summary statistics of completed QC files are generated using FastQC. An example of the structure of the command used is: 

	fastqc -t 8 sampleID_*.fastq.gz

FastQC output files are then combined into a single report using MultiQC, with the sampleID being added to all multiQC outputs at the completion of processing. An example of the structure of the command used is:

	multiqc ./sampleID_* -o sampleID_MGQC_stats

QC statistic files are provided to the Australian microbiome web portal on a per sample basis in a zip package. Each zip package contains an additional file containing md5 checksums for each QC metagenome file. Naming convention for the zip packet is as below:

	sampleID_MGSD_CSIRO.qc.stats.zip

### References

B. Bushnell. BBMap: A Fast, Accurate, Splice-Aware Aligner. United States: N. p., 2014. https://sourceforge.net/projects/bbmap 

S. Andrews. FastQC: a quality control tool for high throughput sequence data (2010). Available online at: http://www.bioinformatics.babraham.ac.uk/projects/fastqc 

P. Ewels, M. Magnusson, S. Lundin and M. Käller. MultiQC: Summarize analysis results for multiple tools and samples in a single report (2016). Bioinformatics 32: 3047–3048
