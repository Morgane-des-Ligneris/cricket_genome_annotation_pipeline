## Teleogryllus occipitalis

**Retriving genome** 

I am using the assembly available on [NCBI](https://www.ncbi.nlm.nih.gov/genome/?term=txid470939[orgn]). 

```
curl -L 'https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/011/170/035/GCA_011170035.1_Tocci_1.0/GCA_011170035.1_Tocci_1.0_genomic.fna.gz' --output gen_teleogryllus_occipitalis.fa.gz
gzip -d gen_teleogryllus_occipitalis.fa.gz 
sed 's/ /_/g' gen_teleogryllus_occipitalis.fa | sed 's/_Teleogryllus_occipitalis_DNA,//g' | sed 's/,_whole_genome_shotgun_sequence//g' | sed 's/_Teleogryllus_occipitalis_mitochondrial_DNA,_complete_genome//g' > genome_teleogryllus_occipitalis.fa && rm gen_teleogryllus_occipitalis.fa
```
**Run Repeatmodeler and RepeatMasker**

With the Dfam TE Tools Container activated :

```
mkdir database ; cd database 
BuildDatabase -name teleogryllus_occipitalis.DB -engine NCBI ../genome_teleogryllus_occipitalis.fa
RepeatModeler -database teleogryllus_occipitalis.DB -engine NCBI -pa 30
cd .. 
ln -s ./database/RM_23.MonJun71320472021/consensi.fa.classified # use it with your consenti.fa.classified path
RepeatMasker -pa 30 -lib consensi.fa.classified genome_teleogryllus_occipitalis.fa -xsmall
```

**Run VARUS**  TO DO 
Paste the file `VARUSparameters.txt`, don't forget to replace the `Runlist.txt` if you want to align all the RNAseq data available for the five species.
```
mkdir varus ; cd varus 
# create or paste VARUSparameters.txt
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics --latinGenus=teleogryllus --latinSpecies=occipitalis --speciesGenome=../genome_teleogryllus_occipitalis.fa.masked --nocreateRunList
```

**Run BRAKER2** 

First test with arthropoda proteins and RNAseq librairies from the species studied here. 
```
cd .. ; mkdir braker2 ; cd braker2 ;
perl /home/ubuntu/data/mydatalocal/tools/BRAKER/scripts/braker.pl --species=occipitalis --genome=../genome_teleogryllus_occipitalis.fa.masked --bam=../varus2/teleogryllus_occipitalis/VARUS.bam --prot_seq=../../proteins/proteins.fasta --softmasking --cores=8 --etpmode --MAKEHUB_PATH=/mnt/mydatalocal/tools/MakeHub/ --ab_initio --makehub --email=morgane.des-ligneris@etu.univ-lyon1.fr
```

**Run BUSCO** done without the correct softmasking, need to be done again once the correct softmasking is finished

```
docker run -u $(id -u) -v $(pwd):/busco_wd ezlabgva/busco:v5.1.3_cv1 busco -m proteins -i ./braker2/braker/proteins.fa -o busco -l arthropoda_odb10 --cpu 8
 
```

RESULTS in `short_summary.specific.arthropoda_odb10.busco_bimaculatus.txt`.

```
# BUSCO version is: 5.1.3 
# The lineage dataset is: arthropoda_odb10 (Creation date: 2020-09-10, number of genomes: 90, number of BUSCOs: 1013)
# Summarized benchmarking in BUSCO notation for file /busco_wd/braker2/braker/proteins.fa
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