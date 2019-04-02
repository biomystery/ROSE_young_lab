ROSE: RANK ORDERING OF SUPER-ENHANCERS
============================================================

ROSE IS RELEASED UNDER THE MIT X11 LICENSE

EXAMPLE DATA AND ADDTIONAL INFORMATION CAN BE FOUND HERE:
http://younglab.wi.mit.edu/super_enhancer_code.html

N.B. LICENSE.txt

For details of this analysis see:

>Master Transcription Factors and Mediator Establish Super-Enhancers at Key Cell Identity Genes 
>Warren A. Whyte, David A. Orlando, Denes Hnisz, Brian J. Abraham, Charles Y. Lin, Michael H. Kagey, Peter B. Rahl, Tong Ihn Lee and Richard A. Young
>Cell 153, 307-319, April 11, 2013

and

>Selective Inhibition of Tumor Oncogenes by Disruption of Super-enhancers 
>Jakob Lovén, Heather A. Hoke, Charles Y. Lin, Ashley Lau, David A. Orlando, Christopher R. Vakoc, James E. Bradner, Tong Ihn Lee, and Richard A. Young
>Cell 153, 320-334, April 11, 2013

Please cite these papers when using this code.

>SOFTWARE AUTHORS: Charles Y. Lin, David A. Orlando, Brian J. Abraham
>CONTACT: young_computation@wi.mit.edu 
>ACKNOWLEDGEMENTS: Graham Ruby
>Developed using Python 2.7.3, R 2.15.3, and SAMtools 0.1.18

**PURPOSE**: To create stitched enhancers, and to separate super-enhancers from typical enhancers using sequencing data (.bam) given a file of previously identified constituent enhancers (.gff)

## 1) PREPARATION/REQUIREMENTS

* .bam files of sequencing reads for factor of interest and control (WCE/IgG recommended).
* .bam files must have chromosome IDs starting with "chr"
* .bam files must be sorted and indexed using SAMtools in order for bamToGFF.py to work. (http://samtools.sourceforge.net/samtools.shtml)
* Code must be run from directory in which it is stored.
* .gff file of constituent enhancers previously identified (gff format ref: https://genome.ucsc.edu/FAQ/FAQformat.html#format3).
* .gff must have the following columns:
  1: chromosome (chr#)
  2: unique ID for each constituent enhancer region
  4: start of constituent
  5: end of constituent
  7: strand (+,-,.)
  9: unique ID for each constituent enhancer region
  NOTE: if value for column 2 and 9 differ, value in column 2 will be used

## 2) CONTENTS

* `ROSE_main.py`: main program
* `ROSE_utils.py`: utility methods
* `ROSE_bamToGFF.py`: calculates density of .bam sequencing reads in .gff regions
* `ROSE_callSuper.R`: ranks regions by their densities, creates a cutoff to separate super-enhancers from typical enhancers
* `ROSE_geneMapper.py`: assigns stitched enhancers to genes
* `annotation/`: Refseq gene tables for genomes MM8,MM9,MM10,HG18,HG19,HG38

In `ROSE_DATA`:

* `example.sh`: sample call of ROSE_main.py
* `data/`: folder containing an example .gff input file and two example .bam files
* `example/`: folder containing example output generated by example.sh


## 3) USAGE

Program is run by calling ROSE_main.py

From within root directory: 
`python ROSE_main.py -g GENOME_BUILD -i INPUT_CONSTITUENT_GFF -r RANKING_BAM -o OUTPUT_DIRECTORY [optional: -s STITCHING_DISTANCE -t TSS_EXCLUSION_ZONE_SIZE -c CONTROL_BAM]`

### Required parameters

* `GENOME_BUILD`: one of hg18, hg19, hg38, mm8, mm9, or mm10 referring to the UCSC genome build used for read mapping 
* `INPUT_CONSTITUENT_GFF`: .gff file (described above) of regions that were previously calculated to be enhancers. I.e. Med1-enriched regions identified using MACS.
* `RANKING_BAM`: .bam file to be used for ranking enhancers by density of this factor. I.e. Med1 ChIP-Seq reads.
* `OUTPUT_DIRECTORY`: directory to be used for storing output.

### Optional parameters:

* `STITCHING_DISTANCE`: maximum distance between two regions that will be stitched together (Default: 12.5kb)
* `TSS_EXCLUSION_ZONE_SIZE`: exclude regions contained within +/- this distance from TSS in order to account for promoter biases (Default: 0; recommended if used: 2500). If this value is 0, will not look for a gene file.
* `CONTROL_BAM`: .bam file to be used as a control. Subtracted from the density of the RANKING_BAM. I.e. Whole cell extract reads.

## 4) CODE PROCEDURE:

`ROSE_main.py` will:

1. format output directory hierarchy
2. Root name of input .gff ([input_enhancer_list].gff) used as naming root for output files.
3. stitch enhancer constituents in INPUT_CONSTITUENT_GFF based on STITCHING_DISTANCE and make .gff and .bed of stitched collection 
4. TSS exclusion, if not zero, is attempted before stitching
5. Names of stitched regions start with number of regions stitched followed by leftmost constituent ID
6. call `bamToGFF.py` to get density of `RANKING_BAM` and `CONTROL_BAM` in stitched regions and constituents
7. Maximum time to wait for `bamToGFF.py` is 12h but can be changed -- quits if running too long
8. call `callSuper.R` to sort stitched enhancers by their background-subtracted density of `RANKING_BAM` and separate into two groups

## 5) OUTPUT:

All file names begin with the root of INPUT_CONSTITUENT_GFF

**OUTPUT_DIRECTORY/gff/**

* `.gff`: copied .gff file of INPUT_CONSTITUENT_GFF - `(chrom, name, [blank], start, end, [blank], [blank], strand, [blank], [blank], name)`
* `STITCHED.gff`: regions created by stitching together INPUT_CONSTITUENT_GFF at STITCHING_DISTANCE - `(chrom, name, [blank], start, end, [blank], [blank], strand, [blank], [blank], name)`. Name is number of constituents stitched together followed by ID of leftmost constituent.

**OUTPUT_DIRECTORY/mappedGFF/:**

* `_MAPPED.gff`: output of bamToGFF using each bam file containing densities of factor in each constituent
(constituent ID, region tested, average read density in units of reads-per-million-mapped per bp of constituent)
* `_STITCHED*_MAPPED.gff`: output of bamToGFF using each bam file containing densities of factor in each stitched enhancer
(stitched enhancer ID, region tested, average read density in units of reads-per-million-mapped per bp of stitched enhancer)

**OUTPUT_DIRECTORY/**:

* `STITCHED_ENHANCER_REGION_MAP.txt`: all densities from bamToGFF calculated in stitched enhancers 
	(stitched enhancer ID, chromosome, stitched enhancer start, stitched enhancer end, number of constituents stitched, rank of RANKING_BAM signal, signal of RANKING_BAM) Signal of RANKING_BAM is density times length. 
* `_AllEnhancers.table.txt`: Rankings and super status for each stitched enhancer
(stitched enhancer ID, chromosome, stitched enhancer start, stitched enhancer end, number of constituents stitched, size of constituents that were stitched together, signal of RANKING_BAM, rank of RANKING_BAM, binary of super-enhancer (1) vs. typical (0)) 
Signal of RANKING_BAM is density times length.
* `_SuperEnhancers.table.txt`: Rankings and super status for super-enhancers 
(stitched enhancer ID, chromosome, stitched enhancer start, stitched enhancer end, number of constituents stitched, size of constituents that were stitched together, signal of RANKING_BAM, rank of RANKING_BAM, binary of super-enhancer (1) vs. typical (0)) 
Signal of RANKING_BAM is density times length.
* `_Enhancers_withSuper.bed`: .bed file to be loaded into the UCSC browser to visualize super-enhancers and typical enhancers.
(chromosome, stitched enhancer start, stitched enhancer end, stitched enhancer ID, rank by RANKING_BAM signal)
* `_Plot_points.png`: visualization of the ranks of super-enhancers and the two groups. Stitched enhancers are ranked by their RANKING_BAM signal and their ranks are along the X axis. Corresponding RANKING_BAM signal on the Y axis.


## NOTES:

* `mapEnhancerFromFactor.py` has a debug mode that can be enabled in the beginning of the main function.
* Enhancers in INPUT_RANKING_GFF may overlap each other in the input.
* This code can be easily parallelized by following the instructions in the main function of mapEnhancerFromFactor around line 369.
* Other gene lists may be added if downloaded from UCSC.
* Code from external sources are also cited in-line.
