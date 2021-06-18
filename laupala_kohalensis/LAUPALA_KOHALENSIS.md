## Laupala kohenlensis

**Retriving genome** 

I am using the assembly available on the [NCBI](https://www.ncbi.nlm.nih.gov/genome/?term=txid109027[orgn]). 
```
curl https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/002/313/205/GCA_002313205.1_ASM231320v1/GCA_002313205.1_ASM231320v1_genomic.fna.gz --output genome_laupala_kohalensis.fa.gz
gzip -d genome_laupala_kohalensis.fa.gz 
sed 's/ /_/g' gen_laupala_kohalensis.fa | sed 's/,_whole_genome_shotgun_sequence//g' | sed 's/_Laupala_kohalensis_isolate_Lakoh051//g' > genome_laupala_kohalensis.fa && rm gen_laupala_kohalensis.fa
```
**Run Repeatmodeler and RepeatMasker** 

```
mkdir database ; cd database
BuildDatabase -name DB_laupala_kohalensis.DB -engine NCBI ../genome_laupala_kohalensis.fa
RepeatModeler -database DB_laupala_kohalensis.DB -engine NCBI -pa 8

cd .. ; ln -s ./database/RM_19.MonMay312019252021/consensi.fa.classified 
RepeatMasker -pa 16 -lib consensi.fa.classified genome_laupala_kohalensis.fa -xsmall
```

**Run VARUS** RUNNING  

The command is the following, don't forget to replace the `Runlist.txt` if you want to align all the RNAseq data available for the five species. 
For this specie we are 'tcheating', because there is no RNAseq data available but I still want to see if RNAseq data from other species is aligning I am putting intentionnaly the wrong name since we are anyway changing the `Runlist.txt` file. 

Before running the following command you also need to had `--limitGenomeGenerateRAM 106616438144` ligne 312 of the `runVarus.pl` file, in order for STAR to be able to index this genome. 

```
mkdir varus ; cd varus 
# create or paste VARUSparameters.txt
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics --latinGenus=gryllus --latinSpecies=bimaculatus --speciesGenome=../genome_laupala_kohalensis.fa.masked --nocreateRunList  
```

**Run BRAKER2**

```
cd .. ; mkdir braker2 ; cd braker2 ;
perl /home/ubuntu/data/mydatalocal/tools/BRAKER/scripts/braker.pl --species=kohenlensis --genome=../genome_laupala_kohalensis.fa.masked --bam=../varus2/gryllus_bimaculatus/VARUS.bam --prot_seq=../../proteins/proteins.fasta --AUGUSTUS_ab_initio --softmasking --cores=1 --etpmode --MAKEHUB_PATH=/mnt/mydatalocal/tools/MakeHub/  --makehub --email=morgane.des-ligneris@etu.univ-lyon1.fr
```