## Teleogryllus oceanicus

I am using the assembly available on the this [platform](http://download.chirpbase.org/v2/features/). 
```
curl http://download.chirpbase.org/v2/sequence/Teleogryllus_oceanicus_Toce1.scaffolds.fa.gz --output genome_teleogryllus_oceanicus.fa.gz
gzip -d gen_teleogryllus_oceanicus.fa.gz 
sed 's/_pilon teleogryllus_oceanicus_toce1_core_36_89_1 scaffold//g' gen_teleogryllus_oceanicus.fa > genome_teleogryllus_oceanicus.fa && rm gen_teleogryllus_oceanicus.fa
```

**Run Repeatmodeler and RepeatMasker** TO DO 

```
BuildDatabase -name teleogryllus_oceanicus.DB -engine NCBI genome_teleogryllus_oceanicus.fa
RepeatModeler -database teleogryllus_oceanicus.DB -engine NCBI -pa 20
```

**Run VARUS** TO DO 

The command is the following, don't forget to replace the `Runlist.txt` if you want to align all the RNAseq data available for the five species.
```
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics \
  --latinGenus=teleogryllus --latinSpecies=oceanicus \
  --speciesGenome=../genome_teleogryllus_oceanicus.fa 
```
