# low depth seq data analysis
accordind to [article](https://pubmed.ncbi.nlm.nih.gov/32274292/), in genome, the highly experssion gene gets the loose physical structure, which indicate that, when cell lysis, this apart of genomic material can be easily digested into small pieces. so when seq the cell-free DNA(cf-DNA), the highly expression part get the low genome coverrage. then the coverage pattern can reveal the epxression pattern. also in pregnant woman, the cfDNA come from the placenta. thus the experssion pattern can help us predict the pregnant complications. 
![cf-dna_pattern](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC7141029/bin/ADVS-7-1901819-g001.jpg)
then for non-invasion prenatal test data. we analysis the target region,like the promoter or the CpG islands, to reveal the placenta expression pattern.
## map the seq data to ref-genome
    bwa aln -n 0 -e 0 -k 0 -t 16 /Users/lave/Documents/analysis/hg38.fa /Users/lave/Documents/analysis/NIPT/SRR6040550_1.fastq > /Users/lave/Documents/analysis/NIPT/SRR6040550_1.sai
    bwa samse -n -1 /Users/lave/Documents/analysis/hg38.fa /Users/lave/Documents/analysis/NIPT/SRR6040550_1.sai /Users/lave/Documents/analysis/NIPT/SRR6040550_1.fastq > /Users/lave/Documents/analysis/NIPT/SRR6040550_1.sam
use bwa aln to map the seq data to ref genome

## sorted the seq data
    samtools view /Users/lave/Documents/analysis/NIPT/SRR6040550_1.sam -bSh > /Users/lave/Documents/analysis/NIPT/SRR6040550_1.bam
    samtools sort -@ 16 /Users/lave/Documents/analysis/NIPT/SRR6040550_1.bam -o /Users/lave/Documents/analysis/NIPT/SRR6040550_1.sorted.bam
    samtools index /Users/lave/Documents/analysis/NIPT/SRR6040550_1.sorted.bam

transfer the sam-file into bam-file, the sorted the data, and creat a index for sorted data.

## filter
    samtools rmdup -s /Users/lave/Documents/analysis/NIPT/SRR6040550_1.sorted.bam /Users/lave/Documents/analysis/NIPT/SRR6040550_1.rmdups.bam
    samtools view -F 4 /Users/lave/Documents/analysis/NIPT/SRR6040550_1.rmdups.bam -bSh > /Users/lave/Documents/analysis/NIPT/SRR6040550_1.final.bam
    samtools index /Users/lave/Documents/analysis/NIPT/SRR6040550_1.final.bam

## correct the reads number with GC content
    /Users/lave/Documents/genomic\ software\ for\ Mac/faToTwoBit /Users/lave/Documents/analysis/hg38.fa  /Users/lave/Documents/analysis/hg38.2bit 
    computeGCBias -b /Users/lave/Documents/analysis/NIPT/SRR6040550_1.final.bam --effectiveGenomeSize 2864785220 -g /Users/lave/Documents/analysis/hg38.2bit -l 200 -o /Users/lave/Documents/analysis/NIPT/SRR6040550_1.freq --region X --biasPlot test_gc.png

## creat the target region data
    bedtools intersect -a /Users/lave/Documents/analysis/NIPT/output/SRR6676163_1.final.bam -b /Users/lave/Documents/analysis/bed/hg38_CpGislands.bed > cpGiland.region.bam

## count reads
    bedtools genomecov -ibam /Users/lave/Documents/analysis/NIPT/output/cpGiland.region.bam -bg  |  awk '$4 > 3' > genomecov.cpg.txt

## nomalize with RPKM method
