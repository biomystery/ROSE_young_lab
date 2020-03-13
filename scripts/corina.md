Super Enhancer calling from ChIP-seq 
----------

Fastq files were mapped to hg19 genome using BWA and the resulted bam
files were filtered with mapq score larger than 30 and deduplicated using
Picard. Peaks were then called on the filtered bam files using MACS2
with option '--nomodel' and with input bam files as background. Broad
peaks were called for H3K27Ac with qvalue cut-off of 0.01. Then the
broad peaks were converted to gff format using awk command. Finally,
the converted gff files together with bam files for H3K27Ac and input
were input into ROSE
([PMID:23582322](https://www.ncbi.nlm.nih.gov/pubmed/23582322),[PMID:23582323](https://www.ncbi.nlm.nih.gov/pubmed/23582323))   
to call super enhancer with default
parameters,i.e. `-STITCHING_DISTANCE 12500  -TSS_EXCLUSION_ZONE_SIZE 2500`. 

