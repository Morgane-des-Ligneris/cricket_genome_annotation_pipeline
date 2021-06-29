## Teleogryllus oceanicus

I am using the assembly available on the this [platform](http://download.chirpbase.org/v2/features/). 
```
curl http://download.chirpbase.org/v2/sequence/Teleogryllus_oceanicus_Toce1.scaffolds.fa.gz --output genome_teleogryllus_oceanicus.fa.gz
gzip -d gen_teleogryllus_oceanicus.fa.gz 
sed 's/_pilon teleogryllus_oceanicus_toce1_core_36_89_1 scaffold//g' gen_teleogryllus_oceanicus.fa > genome_teleogryllus_oceanicus.fa && rm gen_teleogryllus_oceanicus.fa
```

**Busco on the assembly**
```
docker run -u $(id -u) -v $(pwd):/busco_wd ezlabgva/busco:v5.1.3_cv1 busco -m genome -i genome_teleogryllus_oceanicus.fa -o busco_assembly_oceanicus -l arthropoda_odb10 --cpu 6
```
```
        --------------------------------------------------
        |Results from dataset arthropoda_odb10            |
        --------------------------------------------------
        |C:92.5%[S:90.6%,D:1.9%],F:5.4%,M:2.1%,n:1013     |
        |937    Complete BUSCOs (C)                       |
        |918    Complete and single-copy BUSCOs (S)       |
        |19     Complete and duplicated BUSCOs (D)        |
        |55     Fragmented BUSCOs (F)                     |
        |21     Missing BUSCOs (M)                        |
        |1013   Total BUSCO groups searched               |
        --------------------------------------------------
```

**Run Repeatmodeler and RepeatMasker** 

```
mkdir database ; cd database 
BuildDatabase -name teleogryllus_oceanicus.DB -engine NCBI ../genome_teleogryllus_oceanicus.fa
RepeatModeler -database teleogryllus_oceanicus.DB -engine NCBI -pa 10
cd .. 
ln -s ./database/RM_24.ThuJun31149132021/consensi.fa.classified # use it with your consenti.fa.classified path 
RepeatMasker -pa 16 -lib consensi.fa.classified genome_teleogryllus_oceanicus.fa -xsmall
```

**Run VARUS** RUNNING 

Before running the following command you also need to had `--limitGenomeGenerateRAM 139127187498` ligne 312 of the `runVarus.pl` file, in order for STAR to be able to index this genome. 

The command is the following, don't forget to replace the `Runlist.txt` if you want to align all the RNAseq data available for the five species.

```
mkdir varus ; cd varus 
# create or paste VARUSparameters.txt
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics --latinGenus=teleogryllus --latinSpecies=oceanicus --speciesGenome=../genome_teleogryllus_oceanicus.fa.masked --nocreateRunList
```
