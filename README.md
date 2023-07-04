# Downloading human reference genome (GRCh38) and annotation gff3 file using wget command
wget https://ftp.ensembl.org/pub/release-109/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.chromosome.{1..22}.fa.gz && wget https://ftp.ensembl.org/pub/release-109/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.chromosome.{X..Y}.fa.gz

# Using bowtie2 to map the sgRNA sequences (library.fna)
bowtie2-build --threads 12 Homo_sapiens.GRCh38.[list all chromosomes, comma separated] humanIndex
bowtie2 -p 12 -U library.fa -f -x ./chromosomes/humanIndex -S 01_processing/all_library.sam

# Running annotation
samtools view -S -b all_library.sam > all_library.bam && samtools sort all_library.bam > all_library_sorted.bam
gff2bed < ../../Homo_sapiens.GRCh38.109.chr.gff3 > Human_gff.bed && sort -k1,1 -k2,2n Human_gff.bed > Human_gff_sorted.bed
bedtools intersect -bed -a 02_gff_bam/all_library_sorted.bam -b 02_gff_bam/Human_gff_sorted.bed -wa -wb > 03_bam_bed/intersecting.bed

# Extracting only gene annotations
