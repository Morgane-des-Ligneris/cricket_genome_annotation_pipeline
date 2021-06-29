## Gryllus bimaculatus

**Retriving genome** 

I am using the assembly available on this [platform](http://gbimaculatusgenome.rc.fas.harvard.edu) referenced in the article and on the [NCBI](https://www.ncbi.nlm.nih.gov/assembly/GCA_017312745.1/) 
```
curl https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/017/312/745/GCA_017312745.1_Gbim_1.0/GCA_017312745.1_Gbim_1.0_genomic.fna.gz --output gen_gryllus_bimaculatus.fa.gz
gzip -d gen_gryllus_bimaculatus.fa.gz
sed 's/ /_/g' gen_gryllus_bimaculatus.fa | sed 's/,_whole_genome_shotgun_sequence//g' | sed 's/_Gryllus_bimaculatus_DNA,//g' > genome_gryllus_bimaculatus.fa && rm gen_gryllus_bimaculatus.fa
```

**Busco on the assembly**
```
docker run -u $(id -u) -v $(pwd):/busco_wd ezlabgva/busco:v5.1.3_cv1 busco -m genome -i genome_gryllus_bimaculatus.fa -o busco_assembly_bimaculatus -l arthropoda_odb10 --cpu 6
```
```
        --------------------------------------------------
        |Results from dataset arthropoda_odb10            |
        --------------------------------------------------
        |C:98.9%[S:96.6%,D:2.3%],F:0.5%,M:0.6%,n:1013     |
        |1002   Complete BUSCOs (C)                       |
        |979    Complete and single-copy BUSCOs (S)       |
        |23     Complete and duplicated BUSCOs (D)        |
        |5      Fragmented BUSCOs (F)                     |
        |6      Missing BUSCOs (M)                        |
        |1013   Total BUSCO groups searched               |
        --------------------------------------------------
```

**Run Repeatmodeler and RepeatMasker**

```
mkdir database ; cd database 
BuildDatabase -name gryllus_bimaculatus.DB -engine NCBI ../genome_gryllus_bimaculatus.fa
RepeatModeler -database gryllus_bimaculatus.DB -engine NCBI -pa 16
cd .. 
ln -s ./database/RM_33.WedJun91310442021/consensi.fa.classified
RepeatMasker -pa 30 -lib consensi.fa.classified genome_gryllus_bimaculatus.fa -xsmall
```

**Run VARUS** RUNNING 

VARUS is running with STAR but this genome there need to have a parameter adjusted in order to work. 
The parameter is the following and it iss necessary to add this two parameter in ligne 312 of `runVARUS.pl`.
```--limitGenomeGenerateRAM 37497803477```

The command is then the following, don't forget to add the `Runlist.txt` inside the newly created folder `gryllus_bimaculatus` since the createRunList is disable to put our custom `Runlist.txt` to align all the RNAseq data available for the five species.
```
mkdir varus ; cd varus 
# create or paste VARUSparameters.txt
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics --latinGenus=gryllus --latinSpecies=bimaculatus --speciesGenome=../genome_gryllus_bimaculatus.fa.masked --nocreateRunList
```

**Run BRAKER2** TO DO AGAIN 

Running with arthropoda proteins and bam file alligned against all RNAseq librairies available for this species. RNAseq data from other species was not used because there was already enough librairies for this specie (108 available on the NCBI). (The file VARUS.bam here was copied from another simple run of varus on the specie nampe.)
```
cd .. ; mkdir braker2 ; cd braker2 ;
perl /home/ubuntu/data/mydatalocal/tools/BRAKER/scripts/braker.pl --species=bimaculatus --genome=genome_gryllus_bimaculatus.fa.masked --bam=./varus/VARUS.bam --prot_seq=../proteins/proteins.fasta --softmasking --etpmode --cores=8


cd .. ; mkdir braker2 ; cd braker2 ;
perl /home/ubuntu/data/mydatalocal/tools/BRAKER/scripts/braker.pl --species=bimaculatus --genome=../genome_gryllus_bimaculatus.fa.masked --bam=../varus2/gryllus_bimaculatus/VARUS.bam --prot_seq=../../proteins/proteins.fasta --AUGUSTUS_ab_initio --softmasking --cores=1 --etpmode --MAKEHUB_PATH=/mnt/mydatalocal/tools/MakeHub/  --makehub --email=morgane.des-ligneris@etu.univ-lyon1.fr
```

 WARNING: in file /home/ubuntu/data/mydatalocal/tools/BRAKER/scripts/braker.pl at line 1341
file /mnt/mydatalocal/cricket_genome_annotation_pipeline/gryllus_bimaculatus/braker2/braker/genome.fa contains a highly fragmented assembly (47877 scaffolds). This may lead to problems when running AUGUSTUS via braker in parallelized mode. You set --cores=8. You should run braker.pl in linear mode on such genomes, though (--cores=1).

**Run BUSCO** 

```
docker run -u $(id -u) -v $(pwd):/busco_wd ezlabgva/busco:v5.1.3_cv1 busco -m proteins -i ./braker/augustus.hints.aa -o busco_bimaculatus -l arthropoda_odb10 --cpu 6
```
Fisrt done without the correct softmasking, need to be done again once the correct softmasking is finished
RESULTS in `short_summary.specific.arthropoda_odb10.busco_bimaculatus.txt`.

```
        --------------------------------------------------
        |Results from dataset arthropoda_odb10            |
        --------------------------------------------------
        |C:64.1%[S:57.2%,D:6.9%],F:3.0%,M:32.9%,n:1013    |
        |649    Complete BUSCOs (C)                       |
        |579    Complete and single-copy BUSCOs (S)       |
        |70     Complete and duplicated BUSCOs (D)        |
        |30     Fragmented BUSCOs (F)                     |
        |334    Missing BUSCOs (M)                        |
        |1013   Total BUSCO groups searched               |
        --------------------------------------------------
```