# RYVU interview test for sgRNA, mapping to human genome; and annotation to describe associated gene expression on Human Genome

## Overview
This project involves downloading the human reference genome (GRCh38) and annotation gff3 file using the `wget` command from Ensembl. The downloaded human genome GRCh38 file is then indexed and used to map sgRNA sequences using `bowtie2-build` and `bowtie2`.

## Downloading the Reference Genome

The reference genome and annotation gff3 file can be downloaded using the following `wget` commands:

```
wget https://ftp.ensembl.org/pub/release-109/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.chromosome.{1..22}.fa.gz
wget https://ftp.ensembl.org/pub/release-109/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.chromosome.{X..Y}.fa.gz
```

## Mapping sgRNA Sequences

The sgRNA sequences (stored in `library.fna`) can be mapped to the human genome sequence using `bowtie2`. First, the human genome sequence must be indexed using the following command:

```
bowtie2-build --threads 12 Homo_sapiens.GRCh38.[list all chromosomes, comma separated] humanIndex
```

Then, `bowtie2` can be run for the sgRNA sequences using the following command:

```
bowtie2 -p 12 -U library.fa -f -x ./chromosomes/humanIndex -S 01_processing/all_library.sam
```

## Running Annotation

The sequence alignment map (SAM) file is converted to binary format and sorted using `samtools` with the following commands:

```
samtools view -S -b all_library.sam > all_library.bam
samtools sort all_library.bam > all_library_sorted.bam
```

`Bedtools` is used to convert the gff annotation file to bed format using the `gff2bed` command:

```
gff2bed < ../../Homo_sapiens.GRCh38.109.chr.gff3 > Human_gff.bed
sort -k1,1 -k2,2n Human_gff.bed > Human_gff_sorted.bed
```

The intersection of the annotation bed file with the bam file is done using `bedtools` with the following command:

```
bedtools intersect -bed -a 02_gff_bam/all_library_sorted.bam -b 02_gff_bam/Human_gff_sorted.bed -wa -wb > 03_bam_bed/intersecting.bed
```

The intersecting bed file will contain both the library sgRNA identifiers and the gene annotations associated. A custom-made script was used to extract only gene annotations.
