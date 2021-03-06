# ANNOTATION FOR CRICKETS GENOMES

# Contents

[Introduction](#introduction)

[Genome annotation](#genome_annotation)

[BRAKER2 installation](#braker2-installation)

[Other softwares](#other-softwares-to-install)
    
# Introduction 

This document relates the different steps realized exploring the annotation of crickets genome during the internship from 01/05/2021 to 30/06/2021.

The pipeline used is BRAKER2.

To see the steps realized on each crickets just go on the specie folder. Or click on the following links :
- [Acheta domesticus](https://github.com/Morgane-des-Ligneris/cricket_genome_annotation_pipeline/blob/main/acheta_domesticus/ACHETA_DOMESTICUS.md)
- [Gryllus bimaculatus](https://github.com/Morgane-des-Ligneris/cricket_genome_annotation_pipeline/blob/main/gryllus_bimaculatus/GRYLLUS_BIMACULATUS.md)
- [Laupala kohalensis](https://github.com/Morgane-des-Ligneris/cricket_genome_annotation_pipeline/blob/main/laupala_kohalensis/LAUPALA_KOHALENSIS.md)
- [Teleogryllus occipitalis](https://github.com/Morgane-des-Ligneris/cricket_genome_annotation_pipeline/blob/main/teleogryllus_occipitalis/TELEOGRYLLUS_OCCIPITALIS.md)
- [Teleogryllus oceanicus](https://github.com/Morgane-des-Ligneris/cricket_genome_annotation_pipeline/blob/main/teleogryllus_oceanicus/TELEOGRYLLUS_OCEANICUS.md)

Only _T.occipitalis_ was finished during this time of the internship.

# Control quality on genome assembly with busco

The complete BUSCO scores are 45.0% for _A.domesticus_, 98.9% for _G.bimaculatus_, 99.0% for _L.kohalensis_, 97.3% for _T.occipitalis_ and 92.5% for _T.oceanicus_. Assemblies seem to have good quality results in terms of BUSCO completeness score, except for _A.domesticus_ which is fragmented by 38.7% and missing 16.3%. BUSCO scores results are in [Figure 1](#figure).

![Figure 1][figure1]
Figure 1 : BUSCO scores with Arthropoda dataset for each genome’s assembly

[figure1]: busco_results/assembly/busco_figure.png 


# GENOME ANNOTATION 
The steps bellow follow the main steps described for example in the following tutorial : ["Tutorial on how to run run BRAKER2 gene prediction pipeline"](https://bioinformaticsworkbook.org/dataAnalysis/GenomeAnnotation/Intro_to_Braker2.html#gsc.tab=0)

## Steps : 
- Protein database preparation with **ProtHint** (informations bellow)
- Creating repeats library and sofstmasking the genome with **Repeatmodeler and RepeatMasker** 
- Mapping RNAseq data with **VARUS** (see important instructions bellow)
- Running **BRAKER2**
- Checking results with **BUSCO**

Illustration of this steps are summurized in Figure 2 :  

![Figure 2][figure2]  
Figure 2 : Schematic of cricket genome annotation pipeline

[figure2]: docs/pipeline.jpeg

The mode in which BRAKER2 was tested is the pipeline D   
![Figure 3][figure3]

Figure 3 : BRAKER2 pipeline D : training GeneMark-ETP+ supported by RNA-Seq alignment information and information from proteins (proteins can be of any evolutionary distance) Source : https://github.com/Gaius-Augustus/BRAKER/blob/master/docs/figs/braker2_ep_rnaseq.png 

[figure3]: docs/braker2_ep_rnaseq.png

## Protein database preparation

On Braker2, they recommend using OrthoDB for protein information. Since the genome of interest is an insect, as advised by the pipeline ProtHint used with BRAKER2, we can annotate with the information of proteins of the phylum Arthropoda. All the proteins in fasta format of this phylum were gathered in a file. The instructions on how to prepare the input OrthoDB proteins are documented here: https://github.com/gatech-genemark/ProtHint#protein-database-preparation."
```
mkdir proteins ; cd proteins ; 
wget https://v100.orthodb.org/download/odb10_arthropoda_fasta.tar.gz
tar xvf odb10_arthropoda_fasta.tar.gz
cat arthropoda/Rawdata/* > proteins.fasta
```

## Retriving RNAseq libraries using VARUS 

Inside the each species folder, from the description on how to run VARUS it is necessary to have the file `VARUSparameters.txt` in the working directory. You can either copy the file available here [VARUSparameters.txt](https://github.com/Morgane-des-Ligneris/cricket_genome_annotation_pipeline/blob/main/VARUSparameters.txt) create the file and paste the following content : 
```
--batchSize 100000
--blockSize 5000
--components 1
--cost 0.001
--deleteLater 0
--estimator 2
--exportObservationsToFile 1
--exportParametersToFile 1
--fastqDumpCall fastq-dump
--genomeDir ./genome/
--lambda 10.0
--lessInfo 1
--loadAllOnce 0
--maxBatches 500
--mergeThreshold 10
--outFileNamePrefix ./
--pathToParameters ./VARUSparameters.txt
--pathToRuns ./
--pathToVARUS /home/ubuntu/data/mydatalocal/tools/VARUS/Implementation/
--profitCondition 0
--pseudoCount 1
--qualityThreshold 5
--randomSeed 1
--readParametersFromFile 1
--runThreadN 8
--verbosityDebug 1

```

After that you can run the command indicated in each specie sections. Following that, in order to align RNAseq data from all the species of interest here, and not just itself, you have to go inside the newly created folder with the name of the specie and replace the `Runlist.txt` with the file available here [Runlist.txt](https://github.com/Morgane-des-Ligneris/cricket_genome_annotation_pipeline/blob/main/Runlist.txt). It is all the accession from the species selected.
This needs to be done during the indexing of the genome, otherwise it will be too late once the program starts to map the data.  

# BRAKER2 installation
Install BRAKER2 following the instructions on the [gitHub](https://github.com/Gaius-Augustus/BRAKER).

## Tools 
BRAKER2 pipeline requieres several softwares including : 
- [AUGUSTUS](https://github.com/Gaius-Augustus/Augustus)
- [GeneMark-EX](http://exon.gatech.edu/GeneMark/license_download.cgi)
Install GeneMark-EX without forgeting the valid key in your home directory as the BRAKER [README](https://github.com/Gaius-Augustus/BRAKER#genemark-ex) recommand.
- BRAKER2's Perl modules dependencies can be installed with the following commands :
```
# for BRAKER 
sudo apt-get update -y 
sudo apt install perl
sudo apt-get install -y cpanminus 

cpanm File::Spec::Functions
cpanm Hash::Merge
cpanm List::Util
cpanm MCE::Mutex
cpanm Module::Load::Conditional
cpanm Parallel::ForkManager
cpanm YAML
cpanm Scalar::Util::Numeric
cpanm Math::Utils
cpanm threads

sudo apt-get install -y php-posix

# for GeneMark-EX
cpanm MCE::Mutex
cpanm Thread::Queue

# for Augustus 
sudo apt-get install libboost-iostreams-dev
sudo apt-get install zlib1g-dev
sudo apt-get install -y libgsl-dev
sudo apt-get install libboost-all-dev
sudo apt-get install libsuitesparse-dev
sudo apt-get install liblpsolve55-dev
sudo apt-get install libsqlite3-dev
sudo apt-get install libmysql++3v5
sudo apt-get install libbamtools-dev
sudo apt-get install libmysqlclient-dev
sudo apt-get install libmysql++-dev
```
- Python3
- [Bamtools](https://github.com/pezmaster31/bamtools)
- [NCBI BLAST+ or DIAMOND](https://github.com/Gaius-Augustus/BRAKER#ncbi-blast-or-diamond) 
- [ProHints](https://github.com/gatech-genemark/ProtHint)

## Optionnal tools installed 
- Samtools
- Biopython
- cdbfasta


# OTHER SOFTWARES TO INSTALL 
- [RepeatModeler and Repeat Masker](https://github.com/ISUgenomics/bioinformatics-workbook/blob/master/dataAnalysis/ComparativeGenomics/RepeatModeler_RepeatMasker.md). Installed using the [Dfam TE Tools Container](https://github.com/Dfam-consortium/TETools) with [docker](https://www.docker.com). Container activated with ``` /your/path/to/dfam-tetools.sh --docker ```
- [STAR](http://manpages.ubuntu.com/manpages/focal/man1/STAR.1.html)
- [VARUS](https://github.com/Gaius-Augustus/VARUS)
- [SRA-toolkit](https://github.com/ncbi/sra-tools/wiki/02.-Installing-SRA-Toolkit)
- [BUSCO](https://busco.ezlab.org/busco_userguide.html#conda-package) 
- [IGV](https://software.broadinstitute.org/software/igv/)