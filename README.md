# Pipeline for annotating cricket genome

This document relates each step of this pipeline to annotate cricket genomes. 

# Contents

[INTRODUCTION](#introduction)

[PIPELINES](#pipelines) 
- [BRAKER2 INSTALLATION](##braker2_installation)

- [FUNNANOTATE INSTALLATION](##funnanotate_installation)

[OTHER SOFTWARES](#other_softwares_to_install)
    
# INTRODUCTION 

The first pipeline used is BRAKER, the other one will be Funannotate. 

# PIPELINES

## BRAKER2 INSTALLATION 
Install BRAKER2 following the instructions on the [gitHub](https://github.com/Gaius-Augustus/BRAKER).

### Tools 
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

### Optionnal tools installed 
- Samtools
- Biopython
- cdbfasta

## FUNNANOTATE INSTALLATION 

Funannotate is another pipeline that can be installed using conda. 

# OTHER SOFTWARES TO INSTALL 
- [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic)
- [RepeatModeler and Repeat Masker](https://github.com/ISUgenomics/bioinformatics-workbook/blob/master/dataAnalysis/ComparativeGenomics/RepeatModeler_RepeatMasker.md). Installed using the [Dfam TE Tools Container](https://github.com/Dfam-consortium/TETools) with [docker](https://www.docker.com). Container activated with 
```
/home/ubuntu/data/mydatalocal/tools/dfam-tetools.sh --docker
```

# GENOME ANNOTATION USING BRAKER2
The instructions and steps bellow follow the ["Tutorial on how to run run BRAKER2 gene prediction pipeline"](https://bioinformaticsworkbook.org/dataAnalysis/GenomeAnnotation/Intro_to_Braker2.html#gsc.tab=0)
STEPS : 
- CREATING repeats library and sofstmasking the genome with Repeat Masker. 
- Trimming and aligning RNAseq data. 

## Acheta domesticus

**Retriving genome** 
I am using the assembly available on the [NCBI](https://www.ncbi.nlm.nih.gov/genome/?term=txid6997[orgn]). 
```
curl https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/002/313/205/GCA_002313205.1_ASM231320v1/GCA_002313205.1_ASM231320v1_genomic.fna.gz --output gen_acheta_domesticus.fa.gz
gzip -d gen_acheta_domesticus.fa.gz  
sed 's/ /_/g' gen_acheta_domesticus.fa > genome_acheta_domesticus.fa
rm gen_acheta_domesticus.fa
```
**Run Repeatmodeler and RepeatMasker**
```
BuildDatabase -name acheta_domesticus.DB -engine NCBI genome_acheta_domesticus.fa
RepeatModeler -database acheta_domesticus.DB -engine NCBI -pa 16

```

## Gryllus bimaculatus

**Retriving genome** 
I am using the assembly available on this [platform](http://gbimaculatusgenome.rc.fas.harvard.edu). 
```
curl https://gbimaculatusgenome.rc.fas.harvard.edu/session/c044a9e5a74f128dbd50738d0ebe0dbc/download/downloadDataGenome?w= --output genome_gryllus_bimaculatus.fa
```
**Run Repeatmodeler and RepeatMasker**
```
BuildDatabase -name gryllus_bimaculatus.DB -engine NCBI genome_gryllus_bimaculatus.fa
RepeatModeler -database gryllus_bimaculatus.DB -engine NCBI -pa 20
```

## Laupala kohenlensis

**Retriving genome** 
I am using the assembly available on the [NCBI](https://www.ncbi.nlm.nih.gov/genome/?term=txid109027[orgn]). 
```
curl https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/002/313/205/GCA_002313205.1_ASM231320v1/GCA_002313205.1_ASM231320v1_genomic.fna.gz --output genome_laupala_kohalensis.fa.gz
gzip -d genome_laupala_kohalensis.fa.gz 
```
**Run Repeatmodeler and RepeatMasker**
```
BuildDatabase -name DB_laupala_kohalensis -engine NCBI genome_laupala_kohalensis.fa
RepeatModeler -database DB_laupala_kohalensis -engine NCBI -pa 20

```

## Teleogryllus occipitalis

**Retriving genome** 
I am using the assembly available on the to [DNA Data Bank of Japan (DDBJ)](https://www.ddbj.nig.ac.jp/news/en/2020-02-27-e.html). 
```
curl -L 'http://getentry.ddbj.nig.ac.jp/getentry/na/BLKR01000001-BLKR01019865/?format=fasta&filetype=gz&trace=true&show_suppressed=false&limit=' --output gen_teleogryllus_occipitalis.fa.gz
gzip -d gen_teleogryllus_occipitalis.fa.gz 
sed 's/ /_/g' gen_teleogryllus_occipitalis.fa > genome_teleogryllus_occipitalis.fa
rm gen_teleogryllus_occipitalis.fa
```
**Run Repeatmodeler and RepeatMasker**
```
BuildDatabase -name teleogryllus_occipitalis.DB -engine NCBI genome_teleogryllus_occipitalis.fa
RepeatModeler -database teleogryllus_occipitalis.DB -engine NCBI -pa 20
ln -s RM_18.SatMay221904112021/consensi.fa.classified 
RepeatMasker -pa 20 -gff -lib RM_18.SatMay221904112021/consensi.fa.classified genome_teleogryllus_occipitalis.fa
```

## Teleogryllus oceanicus

I am using the assembly available on the this [platform](http://download.chirpbase.org/v2/features/). 
```
curl http://download.chirpbase.org/v2/sequence/Teleogryllus_oceanicus_Toce1.scaffolds.fa.gz --output genome_teleogryllus_oceanicus.fa.gz
gzip -d genome_teleogryllus_oceanicus.fa.gz 
```
**Run Repeatmodeler and RepeatMasker**
```
BuildDatabase -name teleogryllus_oceanicus.DB -engine NCBI genome_teleogryllus_oceanicus.fa
RepeatModeler -database teleogryllus_oceanicus.DB -engine NCBI -pa 20
```



