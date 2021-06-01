## Teleogryllus occipitalis

**Retriving genome** 

I am using the assembly available on the to [DNA Data Bank of Japan (DDBJ)](https://www.ddbj.nig.ac.jp/news/en/2020-02-27-e.html). 
```
curl -L 'http://getentry.ddbj.nig.ac.jp/getentry/na/BLKR01000001-BLKR01019865/?format=fasta&filetype=gz&trace=true&show_suppressed=false&limit=' --output gen_teleogryllus_occipitalis.fa.gz
gzip -d gen_teleogryllus_occipitalis.fa.gz 
sed 's/ /_/g' gen_teleogryllus_occipitalis.fa | sed 's/_Teleogryllus_occipitalis_DNA,//g' | 's/|/_/g'> genome_teleogryllus_occipitalis.fa
rm gen_teleogryllus_occipitalis.fa
```

**Run Repeatmodeler and RepeatMasker**

```
mkdir database ; cd database 
BuildDatabase -name teleogryllus_occipitalis.DB -engine NCBI ../genome_teleogryllus_occipitalis.fa
RepeatModeler -database teleogryllus_occipitalis.DB -engine NCBI -pa 20

cd .. 
ln -s ../database/RM_47112.SunMay231510042021/consensi.fa.classified
RepeatMasker -pa 20 -gff -lib consensi.fa.classified genome_teleogryllus_occipitalis.fa
```

**Run VARUS**  

The command is the following, don't forget to replace the `Runlist.txt` if you want to align all the RNAseq data available for the five species.
```
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics \
  --latinGenus=teleogryllus --latinSpecies=occipitalis \
  --speciesGenome=../genome_teleogryllus_occipitalis.fa 
```

**Run BRAKER2** RUNNING

First test with arthropoda proteins and RNAseq librairies from the species studied here. 
```
cd .. ; mkdir braker2 ; cd braker2 ;
perl /home/ubuntu/data/mydatalocal/tools/BRAKER/scripts/braker.pl --species=occipitalis --genome=../genome_teleogryllus_occipitalis.fa.masked --bam=../varus2/teleogryllus_occipitalis/VARUS.bam --prot_seq=../../proteins/proteins.fasta --softmasking --cores=8 --etpmode 
```
