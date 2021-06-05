## Gryllus bimaculatus

**Retriving genome** 

I am using the assembly available on this [platform](http://gbimaculatusgenome.rc.fas.harvard.edu) referenced in the article. 
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

**Run BRAKER2** 

Running with arthropoda proteins and bam file alligned against all RNAseq librairies available for this species. RNAseq data from other species was not used because there was already enough librairies for this specie (108 available on the NCBI). (The file VARUS.bam here was copied from another simple run of varus on the specie nampe.)
```
cd .. ; mkdir braker2 ; cd braker2 ;
perl /home/ubuntu/data/mydatalocal/tools/BRAKER/scripts/braker.pl --species=bimaculatus --genome=genome_gryllus_bimaculatus.fa.masked --bam=./varus/VARUS.bam --prot_seq=../proteins/proteins.fasta --softmasking --etpmode --cores=8
```

**Run BUSCO** DONE

```
docker run -u $(id -u) -v $(pwd):/busco_wd ezlabgva/busco:v5.1.3_cv1 busco -m proteins -i ./braker/proteins.fa -o busco_bimaculatus -l arthropoda_odb10 --cpu 8
```

RESULTS in `short_summary.specific.arthropoda_odb10.busco_bimaculatus.txt`.

```
# BUSCO version is: 5.1.3 
# The lineage dataset is: arthropoda_odb10 (Creation date: 2020-09-10, number of genomes: 90, number of BUSCOs: 1013)
# Summarized benchmarking in BUSCO notation for file /busco_wd/braker/proteins.fa
# BUSCO was run in mode: proteins

	***** Results: *****

	C:100.0%[S:2.0%,D:98.0%],F:0.0%,M:0.0%,n:1013	   
	1013	Complete BUSCOs (C)			   
	20	Complete and single-copy BUSCOs (S)	   
	993	Complete and duplicated BUSCOs (D)	   
	0	Fragmented BUSCOs (F)			   
	0	Missing BUSCOs (M)			   
	1013	Total BUSCO groups searched		   

Dependencies and versions:
	hmmsearch: 3.2
```