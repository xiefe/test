## Overview

Two analysis pipelines were developed to enable RNA-Seq data processing:

1. Gene/isoform expression quantification

   The first pipeline generates expression profiles for genesand transcripts. It aligns sequence reads to reference genome and transcriptome with [STAR](), followed by quantification of genes and transcript isoforms with [RSEM]().

   MapSplice was also tested to compare with STAR aligner.Similar results were generated by MapSplice yet with much longer runtime.

2. Gene fusion detection

   The second pipeline identifies fusion genes. It uses STAR chimeric junction result as input for [STAR-Fusion]() to discover fusion genes.

   ​

   ​

## Software installation

### Software folder

Assume all softwares will be downloaded and installed in **Software** folder under my **personal home(/home/fxie/Software)** directory in this tutorial, you can use another folder to store and install the softwares.

```Sh
#Create Software folder
mkdir ~/Software
cd ~/Software
```



### Star

```sh
cd ~/Software
wget https://github.com/alexdobin/STAR/archive/2.5.2b.tar.gz
tar -xzf 2.5.2b.tar.gz
# The executable program STAR is under this folder
cd STAR-2.5.2b/bin/Linux_x86_64_static

# Add STAR to system path or make a symbolic link for STAR to be in executable path.
sudo vim /etc/profile # vim ~/.profile instead if you don't have sudo permission.
# Paste the following lines to /etc/profile
STAR="/home/fxie/Software/STAR-2.5.2b/bin/Linux_x86_64_static"
PATH="$STAR:$PATH"
```



### RSEM

```sh
cd ~/Software
wget https://github.com/deweylab/RSEM/archive/master.zip
unzip RSEM-master.zip
cd RSEM-master
make

# Add RSEM to system path
sudo vim /etc/profile
# Paste the following lines to /etc/profile
RSEM="/home/fxie/Software/RSEM-master"
PATH="$RSEM:$PATH"
```

### STAR-Fusion

```Sh
cd ~/Software
## 1. STAR-Fusion
# download and uncompress
wget https://github.com/STAR-Fusion/STAR-Fusion/archive/master.zip
unzip STAR-Fusion-master

# Install Perl modules
perl -MCPAN -e shell
   install Set::IntervalTree
   install DB_File
   install URI::Escape
   
apt-get install perl-doc

# Add STAR-Fusion to system path
sudo vim /etc/profile
# Paste the following lines to /etc/profile
STAR_FUSION="/home/fxie/Software/STAR-Fusion-master"
PATH="$STAR_FUSION:$PATH"

## 2. FusionFilter
cd STAR-Fusion-master/FusionFilter
wget https://github.com/FusionFilter/FusionFilter/archive/v0.3.0.tar.gz
tar -zxvf v0.3.0.tar.gz

# Add FusionFilter & utils to system path
sudo vim /etc/profile
# Paste the following lines to /etc/profile
#FusionFilter
FUSIONFILTER="/home/fxie/Software/STAR-Fusion-master/FusionFilter/FusionFilter-0.3.0"
PATH="$FUSIONFILTER:$PATH"
#FusionFilter utils
FUSIONFILTER_Utils="/home/fxie/Software/STAR-Fusion-master/FusionFilter/FusionFilter-0.3.0/util"
PATH="$FUSIONFILTER_Utils:$PATH"
```



### Bowtie

bowtie-build is need to run the genome library preparation of FusionFilter (prep_genome_lib.pl).

```Sh
cd ~/Software
wget  https://sourceforge.net/projects/bowtie-bio/files/bowtie/1.2.1.1/bowtie-1.2.1.1-linux-x86_64.zip
unzip bowtie-1.2.1.1-linux-x86_64.zip

# Add bowtie to system path
sudo vim /etc/profile
BOWTIE="/home/fxie/Software/bowtie-1.2.1.1"
PATH="$BOWTIE:$PATH"
```



### RepeatMasker

RepeatMasker is a program that screens DNA sequences for interspersed repeats and low complexity DNA sequences.

This tool is used for masking the repeat sequences in DNA and the generated output file was further processed for one of the **input file** of **STAR-Fusion** program.

The installation of RepeatMasker contains three parts: **RepeatMasker, trf & RMBlast, configure**

* RepeatMasker

```Sh
cd ~/Software
wget http://www.repeatmasker.org/RepeatMasker-open-4-0-7.tar.gz
tar -zxf RepeatMasker-open-4-0-7.tar.gz

# Add RepeatMasker to system path
sudo vim /etc/profile
# Paste the following lines to /etc/profile
REPEATMASKER="/home/fxie/Software/RepeatMasker"
PATH="$REPEATMASKER:$PATH"

## RepBase
# Download RepBase (RepBaseRepeatMaskerEdition-20170127.tar.gz), registration is required to get the source which may take a few days. This file has been downloaded and can be found at 192.168.0.185:/mnt/2a9a7cd9-d1ff-4727-9ed0-f62499daa579/rnaseq/RepeatMasker/RepBaseRepeatMaskerEdition-20170127.tar.gz

# go to the folder where the RepBaseRepeatMaskerEdition-20170127.tar.gz saved
tar -zxf RepBaseRepeatMaskerEdition-20170127.tar.gz #a Library folder
#place files under Libraries
mv ./Libraries/* ~/Software/RepeatMasker/Libraries 
```

To configure the RepeatMasker and make RepeatMasker usable, two other programs need to be installed: [trf](http://tandem.bu.edu/trf/trf.download.html), [RMBlast](http://www.repeatmasker.org/RMBlast.html)

* trf & RMBlast

```Sh
cd ~/Software
## 1. trf
# Download trf from http://tandem.bu.edu/trf/trf.download.html.
# rename trf409.linux64 to trf and mv to the RepeatMasker folder
mv trf409.linux64 RepeatMasker/trf

## 2. RMBlast
# 2.1 ncbi-blast-2.6.0+-src.tar.gz
wget ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/2.6.0/ncbi-blast-2.6.0+-src.tar.gz
# 2.2 patch for blast, isb-2.6.0+-changes-vers2.patch.gz
wget http://www.repeatmasker.org/isb-2.6.0+-changes-vers2.patch.gz
tar -zxvf ncbi-blast-2.6.0+-src.tar.gz
gunzip isb-2.6.0+-changes-vers2.patch.gz
# patch
cd ncbi-blast-2.6.0+-src
patch -p1 < ../isb-2.6.0+-changes-vers2.patch

# build
cd ncbi-blast-2.6.0+-src/c++
./configure --with-mt --prefix=/usr/local/rmblast --without-debug
make
make install # may get some errors, but it does’t matter.
```

* Configure Repeatmasker

```Sh
cd ~/Software
cd RepeatMasker
perl ./configure

##-------------------Following lines are the processing of configure----------------
RepeatMasker Configuration Program

<PRESS ENTER TO CONTINUE> # Press Enter

**PERL INSTALLATION PATH**
Enter path [ /usr/local/bin/perl ]: # Press Enter or input another perl path

**REPEATMASKER INSTALLATION PATH**
Enter path [ /home/fxie/Software/RepeatMasker ]: # Press Enter

**TRF INSTALLATION PATH**
Enter path [ ]: /home/fxie/Software/RepeatMasker

Add a Search Engine:
Enter Selection: 2

**RMBlast (rmblastn) INSTALLATION PATH**
Enter path [ ]: /home/fxie/Software/ncbi-blast-2.6.0+-src/c++/ReleaseMT/bin

Do you want RMBlast to be your default
search engine for Repeatmasker? (Y/N) [ Y ]: # Press Enter

Add a Search Engine:
Enter Selection: 5
```



## Analysis pipeline

### 1. Gene/isoform expression quantification

### 1.1 STAR : reads alignment

#### Generate index

```Sh
cd ~/Software/STAR-2.5.2b
mkdir genomeDir150Hg19 genomeFastaFiles sjdbGTFfile
cd genomeFastaFiles
wget ftp://ftp.ensembl.org/pub/grch37/release-87/fasta/homo_sapiens/dna/Homo_sapiens.GRCh37.dna_sm.primary_assembly.fa.gz
gunzip Homo_sapiens.GRCh37.dna_sm.primary_assembly.fa.gz
cd ..

cd sjdbGTFfile
wget ftp://ftp.ensembl.org/pub/grch37/release-87/gtf/homo_sapiens/Homo_sapiens.GRCh37.87.gtf.gz
gunzip homo_sapiens/Homo_sapiens.GRCh37.87.gtf.gz
cd ..

STAR --runThreadN 40\
 --runMode genomeGenerate\
 --genomeDir genomeDir150Hg19\
 --genomeFastaFiles genomeFastaFiles/Homo_sapiens.GRCh37.dna_sm.primary_assembly.fa\
 --sjdbGTFfile sjdbGTFfile/Homo_sapiens.GRCh37.87.gtf\
 --sjdbOverhang 150D
```



#### Generate alignments

```Sh
cd ~/Software/STAR-2.5.2b
STAR --runThreadN 40\
 --genomeDir genomeDir150Hg19\
 --readFilesIn <fastq file1> <fastq file2>\ #R1.fastq R2.fastq
 --quantMode GeneCounts TranscriptomeSAM\
 --chimSegmentMin 12\
 --chimJunctionOverhangMin 12\
 --alignSJDBoverhangMin 10\
 --alignMatesGapMax 200000\
 --alignIntronMax 200000\
 --chimSegmentReadGapMax 3\
 --alignSJstitchMismatchNmax 5 -1 5 5\
 --outSAMtype BAM SortedByCoordinate\
 --outFileNamePrefix <outDir>\
 --outReadsUnmapped Fastx\
 --outSAMunmapped Within\
 --outFilterMatchNminOverLread 0\
 --outFilterScoreMinOverLread 0
```

**NOTES:

1. The following parameters are recommended bySTAR-Fusion for chimeric alignments.

    --chimSegmentMin 12\

    --chimJunctionOverhangMin 12\

    --alignSJDBoverhangMin 10\

    --alignMatesGapMax 200000\

    --alignIntronMax 200000\

    --chimSegmentReadGapMax 3\

    --alignSJstitchMismatchNmax 5 -1 5 5\

   ​

2. These two parameters turn offthe short alignment filters. The short alignment filters left significantamount of reads unmapped. Turning off these filters rescue all reads filteredout due to short alignment.

   --outFilterMatchNminOverLread0\

   --outFilterScoreMinOverLread 0

#### output

1. Log files

2. SAM/BAM

   Aligned.out.sam: alignments of reads togenome. Bam or sorted Bam file can be generated if specified in the parameter.

   Aligned.toTranscriptome.out.bam: alignments of reads totranscripts. This is the input file for RSEM.

3. Chimeric junction

4. SJ.out.tab

   SJ.out.tab contains high confidence collapsed splice junctions in tab-delimited format. Note thatSTAR defines the junction start/end as intronic bases, while many other software define them asexonic bases. The columns have the following meaning:

   column 1: chromosome

   column 2: first base of the intron (1-based)

   column 3: last base of the intron (1-based)

   column 4: strand (0: undefined, 1: +, 2: -)

   column 5: intron motif: 0: non-canonical; 1: GT/AG, 2: CT/AC, 3: GC/AG, 4: CT/GC, 5:AT/AC, 6: GT/AT

   column 6: 0: unannotated, 1: annotated (only if splice junctions database is used)

   column 7: number of uniquely mapping reads crossing the junction

   column 8: number of multi-mapping reads crossing the junction

   column 9: maximum spliced alignment overhang

   ​

### 1.2 RSEM, expression quantification

#### Generate Indices

```Sh
cd ~/Software/RSEM-master
rsem-prepare-reference --gtf ../STAR-2.5.2b/sjdbGTFfile/Homo_sapiens.GRCh37.87.gtf ../STAR-2.5.2b/genomeFastaFiles/Homo_sapiens.GRCh37.dna.primary_assembly.fa ref/hg19
```

#### Calculate expression

```Sh
rsem-calculate-expression --paired-end -p 40 –alignments < Aligned.toTranscriptome.out.bam> ref/hg19 <outputPrefix>
# Aligned.toTranscriptome.out.bam is the output of STAR
```

#### output

1. <prefix>.genes.results

   Gene level expression profile, listing expected count, TPM,and FPKM for genes.

2. <prefix>.isoforms.results

   Transcript level expression profile, listing expected count,TMP, and FPKM for transcripts. For each gene, the expected count is the sum ofall transcripts’ counts.

   ​

### 2. Gene fusion detection

### STAR-Fusion

#### Generate indexes

```Sh
## 1 Extract cDNA sequences
cd ~/Software/STAR-2.5.2b
gtf_file_to_cDNA_seqs.pl sjdbGTFfile/Homo_sapiens.GRCh37.87.gtf genomeFastaFiles/Homo_sapiens.GRCh37.dna.primary_assembly.fa > genomeFastaFiles/cDNA_seqsGRCh37.fa

## 2 Mask repetitive regions within cDNAs
RepeatMasker -pa 6 -s -species human -xsmall genomeFastaFiles/cDNA_seqsGRCh37.fa

## 3 All-vs-all BLASTN search
cd genomeFastaFiles
# make the cDNA_seqs.fa file blastable
makeblastdb -in cDNA_seqsGRCh37.fa.masked -dbtype nucl
# perform the blastn search, (takes long time, about 5hs)
blastn -query cDNA_seqsGRCh37.fa -db cDNA_seqsGRCh37.fa.masked \
       -max_target_seqs 10000 -outfmt 6 \
       -evalue 1e-3 -lcase_masking \
       -num_threads <num.threads> \
       -word_size 11  >  blast_pairs.outfmt6

## 4 Prepare the custom FusionFilter Dataset
cd ~/Software/STAR-Fusion-master
mkdir CTAT_resource_lib
mkdir CTAT_resource_lib/GRCh37_ensembl87

prep_genome_lib.pl \
 --genome_fa ../STAR-2.5.2b/genomeFastaFiles/Homo_sapiens.GRCh37.dna.primary_assembly.fa \
 --gtf ../STAR-2.5.2b/sjdbGTFfile/Homo_sapiens.GRCh37.87.gtf \
 --blast_pairs ../STAR-2.5.2b/genomeFastaFiles/blast_pairs.gene_syms.outfmt6.gz \
 --cdna_fa ../STAR-2.5.2b/genomeFastaFiles/cDNA_seqsGRCh37.fa
```



#### Fusion Detection

There are two ways to detect fusion:

1. Start with FASTQ files, using prebuilt FusionFilter data sources 


2. Start with STAR chimeric alignment output, using custom built FusionFilter data sources

**Start with FASTQ files**

```Sh
#download the CTAT_lib
cd ~/Software/STAR-Fusion-master/CTAT_resource_lib
Wget https://data.broadinstitute.org/Trinity/CTAT_RESOURCE_LIB/__pre_July202017/GRCh37_gencode_v19_CTAT_lib_July272016_prebuilt.tar.gz
tar -zxvf GRCh37_gencode_v19_CTAT_lib_July272016_prebuilt.tar.gz

#detect fusion
cd ~/Software/STAR-Fusion-master/
STAR-Fusion --genome_lib_dir CTAT_resource_lib/GRCh37_gencode_v19_CTAT_lib_July272016_prebuilt --left_fq <R1.fastq> --right_fq <R2.fastq> --output_dir <outputDirectory>
```



**Start with STAR chimeric alignment output**

```Sh
#detect fusion
cd ~/Software/STAR-Fusion-master/
STAR-Fusion --genome_lib_dir CTAT_resource_lib/GRCh37_ensembl87/ -J <outputDirectory/Chimeric.out.junction> --output_dir <outputDirectory>
#Chimeric.out.junction is generated bt STAR
```



#### output

The output from STAR-Fusion is found as a tab-delimited file named 'star-fusion.fusion_predictions.tsv', along with an abridged version that excludes the identification of the evidence fusion reads and called 'star-fusion.fusion_predictions.tsv', with the following format:

```Sh
#FusionName     JunctionReadCount       SpanningFragCount       SpliceType      LeftGene        LeftBreakpoint  RightGene       RightBreakpoint LargeAnchorSupport      LeftBreakDinuc  LeftBreakEntropy        RightBreakDinuc RightBreakEntropy       FFPM
THRA--THRA1/BTR 27      93      ONLY_REF_SPLICE THRA^ENSG00000126351.12 chr17:40086853:+        THRA1/BTR^ENSG00000235300.4     chr17:48294347:+        YES_LDAS        GT      1.8892  AG      1.9656  23875.8456
THRA--THRA1/BTR 5       93      ONLY_REF_SPLICE THRA^ENSG00000126351.12 chr17:40086853:+        THRA1/BTR^ENSG00000235300.4     chr17:48307331:+        YES_LDAS        GT      1.8892  AG      1.4295  19498.6072
ACACA--STAC2    12      52      ONLY_REF_SPLICE ACACA^ENSG00000278540.4 chr17:37122531:-        STAC2^ENSG00000141750.6 chr17:39218173:-        YES_LDAS        GT      1.9656  AG      1.9656  12733.7844
```

The JunctionReads column indicates the number of RNA-Seq fragments containing a read that aligns as a split read at the site of the putative fusion junction.

The SpanningFrags column indicates the number of RNA-Seq fragments that encompass the fusion junction such that one read of the pair aligns to a different gene than the other paired-end read of that fragment.



## Reference

[STAR Paper](https://academic.oup.com/bioinformatics/article/29/1/15/272537/STAR-ultrafast-universal-RNA-seq-aligner)

[STAR Github](https://github.com/alexdobin/STAR)

[RSEM Paper](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-12-323)

[RSEM Github](https://github.com/deweylab/RSEM/)

[STAR Fusion Paper](http://www.biorxiv.org/content/early/2017/03/24/120295)

[STAR Fusion Github](https://github.com/STAR-Fusion/STAR-Fusion/wiki)

[Repeatmasker](http://www.repeatmasker.org/)

[RMBlast](http://www.repeatmasker.org/RMBlast.html)



## Test runs and Conclusions (Originally from Jane)

Three test sets: 

1.    Two RNA-Seq experiments were performed for VCaPcell line, one is with strand-specific library preparation, and the other iswith non-strand-specific preparation. For each experiment, index-specific subsets of FASTQ files were concatenated to generate read1 and read2 FASTQfiles. 
2.    Previously processed BAM file from RT+enrichmentassay with VCaP cell line. FASTQ files were extracted with samtools followed by sorting of the reads for R1.fastq and R2.fastq to have matching read names inorder.

Main conclusions and future work:

1. STAR: with default parameters, significant amounts of reads were unmapped due to the short alignment filter: ~40% for RNA-Seq experiments and ~80% for RT+enrichment assay.

2.    STAR: with short alignment filters off, AR v7junction counts (109) matches Predicine in-house bioinformatics pipeline result, greatly increased from count of only 8 with default parameters.

3.    STAR: significantly more splice junctions are found with strand-specific RNA-Seq data (~198K) than the non-strand-specificassay (~120K), either with default parameters or with short alignment filters off. A closed-up examination of AR junctions revealed that strand-specificassay data lead to ARv7 junction identification while non-strand-specific assaydata failed to discover ARv7.

4.    STAR: processing results are similar usingGRCh37 or GRCh38 genome builds.

5.    STAR: completely turning off the short alignment filter may introduce false positives. When paired with Predicine in-house pipeline, specific filters will be in place to eliminate false positives. However, for general RNA-Seq data processing, parameter tuning would be helpful to define the parameter set that optimizes the mapping percentage with minimum false positives.

6.    MapSplice was run on non-strand-specific assaydata. Results appear to be similar to those from STAR, yet with many hours of additional runtime.

7.    RSEM uses expectation maximization algorithm to model the distribution of transcripts given the read alignments to transcriptome. The transcript counts add up to the corresponding gene count. Significant counts for a transcript with no transcript specific splice junctions found by STAR can still be included in the RSEM result.

8.    STAR-Fusion recommended chimera detection parameters for STAR aligner lead to more chimeric junction alignments.

9.    Surprisingly, with prebuilt STAR index and STAR-Fusion defined STAR aligner command, no AR junctions were reported in SJ.out.tab for all three test tests.

10.    Final fusion genes list is similar with custom built data source as the one withprebuilt data source:

       a.    RNA-Seq non-strand-specific: no result.

       b.    RNA-Seq strand-specific: two TMPRSS2-ERG fusion products,plus 11 others with custom built data source; two TMPRSS2-ERG fusion products,plus 17 others with prebuilt data source.

       c.    RT+enrichment: two TMPRSS2-ERG fusion products,plus 2 potentially irrelevant ones. We can filter out MT and scaffold genomesbefore feeding Chimeric.out.junction into STAR-Fusion to avoid such spuriousresults.

Reference: [https://www.ncbi.nlm.nih.gov/pubmed/23615946](https://www.ncbi.nlm.nih.gov/pubmed/23615946). This paper reports the confirmationof 15 fusion candidates.

 

 

