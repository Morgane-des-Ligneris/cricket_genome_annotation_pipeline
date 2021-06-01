## Acheta domesticus

**Retriving genome** 

I am using the assembly available on the [NCBI](https://www.ncbi.nlm.nih.gov/genome/?term=txid6997[orgn]). It is necessary to simplify the genome headers because it can create errors later with BRAKER2. There need to be no spaces or odd caracters, the size need to be less than 50 characters. 
```
curl https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/014/858/955/GCA_014858955.1_NU_Adom_1.1/GCA_014858955.1_NU_Adom_1.1_genomic.fna.gz --output gen_acheta_domesticus.fa.gz
gzip -d gen_acheta_domesticus.fa.gz  
sed 's/ /_/g' gen_acheta_domesticus.fa | sed 's/,_whole_genome_shotgun_sequence//g' | sed 's/_Acheta_domesticus_isolate_YG1991//g' > genome_acheta_domesticus.fa && rm gen_acheta_domesticus.fa
```

**Run Repeatmodeler and RepeatMasker** RUNNING
```
mkdir database ; cd database 
BuildDatabase -name acheta_domesticus.DB -engine NCBI ../genome_acheta_domesticus.fa
RepeatModeler -database acheta_domesticus.DB -engine NCBI -pa 8

# TO DO : 
cd .. 
ln -s ./database/ *to fill* 
RepeatMasker -pa 8 -gff -lib consensi.fa.classified genome_gryllus_bimaculatus.fa
```

**Run VARUS** 
The command is the following, don't forget to replace the `Runlist.txt` if you want to align all the RNAseq data available for the five species.
```
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics \
  --latinGenus=acheta --latinSpecies=domesticus \
  --speciesGenome=../genome_acheta_domesticus.fa 
```
