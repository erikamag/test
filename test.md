#### want a cpm count/exon for regions of homology between wt transcript and assembled mutant transcripts (1. original TSS-Mu termination, 2. Mu TS - end)

get rn21a and rn20h gids from W22 gff3 

subset out gids with Mu transcripts 1-4

	cd /Users/erikamagnusson/Documents/umn.pmb/SpringerLab/genome/Zm00004b.1_W22
	
	# for the genes that I want to get an FPKM/exon or CDS, grep gid_T001 from W22_gene.gff3 file
	
	grep -Ff rn21a-rn20h_wgid.txt Zm-W22-REFERENCE-NRGENE-2.0_Zm00004b.1.gff3 > rntf_w.gff3
	
	grep -e PATTERN1 -e PATTERN2 your_file # if want to match two separate patterns. Want to add longest transcript for one gene and _T001 for all other genes
	
	fgrep -e _T001 -e Zm00004b002134_T003 rntf_w.gff3 > rn21a-20h_transcript.gff3
	
##### now edit this .gff3 to remove introns and remove overlap between exon and CDS (5'UTR and 3' UTR and CDS)

Want to get a cpm count/exon and then I can choose which exon is homologous between WT and mutant transcript
**in .R or xcel**

1. identify the longest transcript/gene for genes I have mutants for
	input .gtf file of each T001 start-end transcript length lines
	end-start and take longest/gene
		* also want to have a T001 file. 
		* Calcuate cpm/exon by: T001 and longest transcript 
	Longest transcript = primary transcript? 
	We do not know this is true unless we compared RNAseq reads of all isoforms/gene
	/Users/erikamagnusson/Documents/umn.pmb/SpringerLab/genome/Zm00004b.1_W22/rntf_w_mRNA_longest_transcript.xlsx #completed in xcel and only one gid has an alternative transcript other than T001 that is longer. 

2. in order to get a scaling factor for normalizing each .bam file from the total number of library reads - sum up the raw read count of all genes for each biological replicate and print to summary table 

**text editor**

3. via text editor, edit W22.gtf file for genes I am calculating cpm/exon and CDS for. Do not want CDS to overlap with exon (5'UTR or 3'UTR). Coordinate of last "exons" - change start to not overlap CDS start

**bedtools in msi**

4. convert .bam file/mutant/biological replicate > .bed files with bedtools
[bamtobed](https://bedtools.readthedocs.io/en/latest/content/tools/bamtobed.html)

		bedtools bamtobed -i reads.bam > read.bed #installed bedtools in conda base
		cd ~/git/nf/rn21a/rnaseq/raw/20_bam
		bedtools bamtobed -i E38_2.bam > ~/git/nf/rn21a/rnaseq/raw/21_bed/E38_2.bed

do not want to use -bedpe option because there are some PE reads that may not be mapped. -bedpe reportes both PE reads on a single line if they are both mappable, but if one PE reads is not mapped, both reads would would have a chr of "" and be considered unmapped. want to include all reads mapped to region even if mate pair did not map

5. then use bedtools intersect to count how many reads intersected with gene coordinates
[bedtools intersect](https://bedtools.readthedocs.io/en/latest/content/tools/intersect.html)

		bedtools intersect -a reads.bed -b reads.gtf -c > read_counts.bed
	
		bedtools intersect -a A.bed -b B.bed -wo # can also use -wo option to report a column after each combination of intersecting “A” and “B” features indicating the amount of overlap in bases pairs that is observed between the two features
	
6. calculate cpm for each read count calculation/CDS or exon by using scalar factor of sum raw read counts/library or /biological replicate
7. get an average cpm for all biological replicates/CDS or exon
