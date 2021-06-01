## Gryllus bimaculatus

**Retriving genome** 

I am using the assembly available on this [platform](http://gbimaculatusgenome.rc.fas.harvard.edu). 
```
curl https://gbimaculatusgenome.rc.fas.harvard.edu/session/c044a9e5a74f128dbd50738d0ebe0dbc/download/downloadDataGenome?w= --output genome_gryllus_bimaculatus.fa
```
**Run Repeatmodeler and RepeatMasker**

```
mkdir database ; cd database 
BuildDatabase -name gryllus_bimaculatus.DB -engine NCBI ../genome_gryllus_bimaculatus.fa
RepeatModeler -database gryllus_bimaculatus.DB -engine NCBI -pa 20

cd .. 
ln -s ./database/RM_23.FriMay281932292021/consensi.fa.classified
RepeatMasker -pa 20 -gff -lib consensi.fa.classified genome_gryllus_bimaculatus.fa
```

**Run VARUS**

The command is the following, for this one maybe this is not necessary to replace  the `Runlist.txt` because there is 108 RNAseq librairies available already. 
```
mkdir varus ; cd varus
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics \
  --latinGenus=gryllus --latinSpecies=bimaculatus \
  --speciesGenome=../genome_gryllus_bimaculatus.fa 
```

**Run BRAKER2** RUNNING

Running with arthropoda proteins and bam file alligned against all RNAseq librairies available for this species. RNAseq data from other species was not used because there was already enough librairies for this specie (108 available on the NCBI). (The file VARUS.bam here was copied from another simple run of varus on the specie nampe.)
```
cd .. ; mkdir braker2 ; cd braker2 ;
perl /home/ubuntu/data/mydatalocal/tools/BRAKER/scripts/braker.pl --genome=../genome_gryllus_bimaculatus.fa.masked --bam=../varus/VARUS.bam --prot_seq=../proteins/proteins.fasta --softmasking --cores=8 --etpmode 
```
