# Aligning E. coli sequencing reads to a reference genome

## Question
Can I take raw paired-end sequencing reads, align them to a reference genome, and filter the alignments by quality using Linux command-line tools?

## Data
- Reads: SRR2589044 (paired-end, E. coli, from the Lenski long-term evolution experiment), downloaded from the European Nucleotide Archive
- Reference: E. coli REL606 genome (NCBI GCA_000017985.1)

## Steps
1. Quality check of the reads: FastQC
2. Trimming: Trimmomatic (adapters and low-quality ends)
3. Indexed the reference: `bwa index ecoli_rel606.fasta`
4. Aligned the reads: `bwa mem -M -t 4 -R '@RG\tID:SRR2589044\tSM:SRR2589044' ecoli_rel606.fasta SRR2589044_1.fastq.gz SRR2589044_2.fastq.gz > SRR2589044.sam`
5. Alignment summary: `samtools flagstat SRR2589044.sam`
6. Filtered to mapping quality 20 or higher: `samtools view -q 20 -b -o SRR2589044.q20.bam SRR2589044.sam`
7. Repeated steps 4 and 6 in one piped command, with no intermediate SAM file: `bwa mem ... | samtools view -q 20 -b -o SRR2589044.piped.bam -`
8. Checked that both methods gave the same number of alignments: `samtools view -c` returned 2182927 for both.

## Results
- 99.45% of reads mapped to the reference
- 95.34% of reads were properly paired
- 0.54% were singletons
- This alignment used the untrimmed reads. BWA soft-clipped adapter sequence on its own, which is why the mapping rate is still high.

## How to run it
Install the tools with conda: `conda create -n ngs -c conda-forge -c bioconda fastqc trimmomatic bwa samtools`, then run the commands in the Steps section from the folder that holds the reference and the reads.
