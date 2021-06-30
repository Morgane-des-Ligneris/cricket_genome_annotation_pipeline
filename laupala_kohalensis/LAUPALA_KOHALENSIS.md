## Laupala kohenlensis

**Retriving genome** 

I am using the assembly available on the [NCBI](https://www.ncbi.nlm.nih.gov/genome/?term=txid109027[orgn]). It is also necessary to simplify the genome headers.
```
curl https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/002/313/205/GCA_002313205.1_ASM231320v1/GCA_002313205.1_ASM231320v1_genomic.fna.gz --output genome_laupala_kohalensis.fa.gz
gzip -d genome_laupala_kohalensis.fa.gz 
sed 's/ /_/g' gen_laupala_kohalensis.fa | sed 's/,_whole_genome_shotgun_sequence//g' | sed 's/_Laupala_kohalensis_isolate_Lakoh051//g' > genome_laupala_kohalensis.fa && rm gen_laupala_kohalensis.fa
```

**Busco on the assembly**
```
busco -m genome -i genome_laupala_kohalensis.fa -o busco_assembly_kohalensis -l arthropoda_odb10 --cpu 6
```
```
        --------------------------------------------------        
        |Results from dataset arthropoda_odb10            |
        --------------------------------------------------        
        |C:99.0%[S:95.8%,D:3.2%],F:0.6%,M:0.4%,n:1013     |
        |1002   Complete BUSCOs (C)                       |
        |970    Complete and single-copy BUSCOs (S)       |  
        |32     Complete and duplicated BUSCOs (D)        |
        |6      Fragmented BUSCOs (F)                     |
        |5      Missing BUSCOs (M)                        |
        |1013   Total BUSCO groups searched               |
        --------------------------------------------------
```

**Run Repeatmodeler and RepeatMasker** 

```
mkdir database ; cd database
BuildDatabase -name DB_laupala_kohalensis.DB -engine NCBI ../genome_laupala_kohalensis.fa

RepeatModeler -database DB_laupala_kohalensis.DB -engine NCBI -pa 8

cd .. 
#change the folowing command with your path to the final consensi.fa.classified file 
ln -s ./database/RM_19.MonMay312019252021/consensi.fa.classified 

RepeatMasker -pa 16 -lib consensi.fa.classified genome_laupala_kohalensis.fa -xsmall
```

**Run VARUS**   

The command is the following, don't forget to replace the `Runlist.txt` if you want to align all the RNAseq data available for the five species. 
For this specie we are 'tcheating', because there is no RNAseq data available but I still want to see if RNAseq data from other species is aligning I am putting intentionnaly the wrong name since we are anyway changing the `Runlist.txt` file. 

Before running the following command you also need to had `--limitGenomeGenerateRAM 106616438144` ligne 312 of the `runVarus.pl` file, in order for STAR to be able to index this genome. 

```
mkdir varus ; cd varus 
# create or paste VARUSparameters.txt
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics --latinGenus=gryllus --latinSpecies=bimaculatus --speciesGenome=../genome_laupala_kohalensis.fa.masked --nocreateRunList  
# paste Runlist.txt inside the newly created gryllus_bimaculatus file
```

**Run BRAKER2**

```
cd .. ; mkdir braker2 ; cd braker2 ;
perl /home/ubuntu/data/mydatalocal/tools/BRAKER/scripts/braker.pl --species=kohenlensis --genome=../genome_laupala_kohalensis.fa.masked --bam=../varus2/gryllus_bimaculatus/VARUS.bam --prot_seq=../../proteins/proteins.fasta --AUGUSTUS_ab_initio --softmasking --cores=1 --etpmode 
```