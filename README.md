# Identifying protein sequences from Interproscan

This project consists of a series of scripts to use [InterProScan](https://github.com/ebi-pf-team/interproscan) to identify two component system (TCS) proteins from prokaryotic genomes and [seqtk](https://github.com/lh3/seqtk) to pull TCS sequences out of whole genomes. We used [Prodigal](https://github.com/hyattpd/Prodigal) to predict putative protein sequences from genomes. 

## Getting Started
If you have are starting with nucleic acid or genome sequences, start with step 1 to generate amino acid fasta (.faa) files and then run these .faa files through Interproscan. If you are running large batches of files on a high-throughput computing cluster, consider [parallelizing Interproscan](https://github.com/rice-crc/interproscan). Otherwise, start with step 2 to identify histidine kinase (HK) and response regulator (RR) sequences.

The scripts in steps 2-4 are used as post-processing and assumes that genomes have already been run through InterProScan. The output from Interproscan should be a .tsv file for each input file (genome) with individual sequence names tagged with InterPro (IPR) signatures. You should also have the putative protein sequence .faa files (again, one for each genome) from Prodigal. Have these files ready to be referenced.

## Dependencies
- [seqtk](https://github.com/lh3/seqtk)
- (if needed for step 1)[Prodigal](https://github.com/hyattpd/Prodigal "Prodigal")

## Identifying & extracting HK and RR sequences
### 1. (If needed) Generating amino acid fasta (.faa) files from nucleic acid fasta files using Prodigal (**prodigal_demmo.sh**)
Converts genome files to predicted proteome files. This project used [Prodigal](https://github.com/hyattpd/Prodigal "Prodigal") to generate predicted protein sequences from metagenome-assembled genomes. prodigal_demmo.sh analyzes batches of nucleic acid files using Prodigal. Run this step before running Interproscan.

Define the paths to your Prodigal installation and your nucleic acid fasta files, then run **prodigal_demmo.sh**.

- Inputs
  - nucleic acid (genome) FASTA files
- Outputs
  - amino acid (proteome) FASTA files
  
### 2. Identifying HK and RR proteins from Interproscan outputs (**IPR_filter.py**)
Counts the number of histidine kinases (HKs) and response regulators (RR) in each amino acid FASTA file.

Define filepaths to .tsv outputs from Interproscan, lists of HK/RR true and false positive IPR signatures, and where you want .csv outputs containing the number of HKs/RRs in each genome file to save. The run **IPR_filter.py**.

- Inputs
  - .tsv Interproscan outputs (containing a list of predicted protein IPR signatures)
  - lists of true positive and false positive HK and RR signatures
- Outputs
  - abundance_counts.csv: summary file containing the number of HKs and RRs identified in each Interproscan .tsv output 
  - HK_IPR_signatures.csv: For each Interproscan file, lists predicted HK sequence IDs and each IPR signature predicted for that HK sequence. The resulting file contains a dictionary of lists in the form of "Interproscan .tsv filename", {HK sequence ID:[list of IPR signatures associated with that HK]}. 
  - RR_IPR_signatures.csv: Same as HK_IPR_signatures, but for RRs

### 3. Extract names of HKs (and IPR signatures associated with each HK) from Interproscan .tsv outputs (**sequence_filter.py**)

Define filepaths to .tsv outputs from Intproscan, and run **sequence_filter.py**.

- Inputs
  - .tsv Interproscan outputs (containing a list of predicted protein IPR signatures)
- Outputs (for each .tsv input file):
  - .csv file containing the name of each HK protein and any other IPR signatures it contains 
  - .lst file containing the name of each HK protein

### 4. Extract HK sequences from .faa files (**extract_sequences.py** and **seqtk_run.sh**)
Define directory paths for inbox containing .faa files from prodigals, directory containing .lst files generated from sequence_filter.py, and run **extract_sequences.py**.

**extract_sequences.py** spawns **seqtk_run.sh** to extract HK seqeunces and renames them to be identifiable by adding the genome ID to the sequence ID (e.g. 11-1 is genome 11, HK 1)

- Inputs:
  - amino acid FASTA (.faa) files (e.g. from prodigal)
  - .lst file containing the name of each HK protein (generated by sequence_filter.py)
- Outputs:
  - .faa file containing all HK sequences identified in the input files (allHKseq.faa) 
  
### HK and RR IPR signature files (contained in HK_RR_IPRsignatures)
These files are referenced by **IPR_filter.py** and **sequence_filter.py**. They contain the IPR signatures for HKs and RRs as well as the false positive IPR signatures for each. 
- HK1ListTSV.txt
- HKFalseListTSV.txt
- RR1ListTSV.txt
- RRFalseListShortTSV.txt

## Open questions
- Figure 6 - the geochemical data we used to generate this figure found in file "120521_fig4.csv" (and "120621_fig3_4_violins.csv"? not sure we used this figure) currently aren't included in supplementary data. Are we citing Maggie's work here? If the geochemical data isn't published elsewhere, we might need to ask Maggie permission to post 
- Still need to add plotting scripts for figs 2, 4, and 5
- Note: I have the input files for figures 1, 3, and 7 in a local copy on my computer (not published to Github yet). I think these are all permutations of data that will be published in the supplement but I want to review it more thoroughly before posting