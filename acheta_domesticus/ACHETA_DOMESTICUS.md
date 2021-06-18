## Acheta domesticus

**Retriving genome** 

I am using the assembly available on the [NCBI](https://www.ncbi.nlm.nih.gov/genome/?term=txid6997[orgn]). It is necessary to simplify the genome headers because it can create errors later with BRAKER2. There need to be no spaces or odd caracters, the size need to be less than 50 characters. 
```
curl https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/014/858/955/GCA_014858955.1_NU_Adom_1.1/GCA_014858955.1_NU_Adom_1.1_genomic.fna.gz --output gen_acheta_domesticus.fa.gz
gzip -d gen_acheta_domesticus.fa.gz  
sed 's/ /_/g' gen_acheta_domesticus.fa | sed 's/,_whole_genome_shotgun_sequence//g' | sed 's/_Acheta_domesticus_isolate_YG1991//g' > genome_acheta_domesticus.fa && rm gen_acheta_domesticus.fa
```

**Busco on the assembly**
```
docker run -u $(id -u) -v $(pwd):/busco_wd ezlabgva/busco:v5.1.3_cv1 busco -m genome -i genome_acheta_domesticus.fa -o busco_assembly_domesticus -l arthropoda_odb10 --cpu 6
```
Results :
```
        --------------------------------------------------
        |Results from dataset arthropoda_odb10            |
        --------------------------------------------------
        |C:45.0%[S:43.9%,D:1.1%],F:38.7%,M:16.3%,n:1013   |
        |456    Complete BUSCOs (C)                       |
        |445    Complete and single-copy BUSCOs (S)       |
        |11     Complete and duplicated BUSCOs (D)        |
        |392    Fragmented BUSCOs (F)                     |
        |165    Missing BUSCOs (M)                        |
        |1013   Total BUSCO groups searched               |
        --------------------------------------------------
```

**Run Repeatmodeler and RepeatMasker** 
```
mkdir database ; cd database 
BuildDatabase -name acheta_domesticus.DB -engine NCBI ../genome_acheta_domesticus.fa
RepeatModeler -database acheta_domesticus.DB -engine NCBI -pa 8
cd .. ; ln -s ./database/RM_18.MonMay310935222021/consensi.fa.classified
RepeatMasker -pa 30 -lib consensi.fa.classified genome_acheta_domesticus.fa -xsmall
```

**Run VARUS**  RUNNING
VARUS is running with STAR but this genome there need to have a parameter adjusted in order to work. 
The parameter is the following in STAR --help : 
```
genomeChrBinNbits           18
    int: =log2(chrBin), where chrBin is the size of the bins for genome storage: each chromosome will occupy an integer number of bins. For a genome with large number of contigs, it is recommended to scale this parameter as min(18, log2[max(GenomeLength/NumberOfReferences,ReadLength)])
```
The number of references is : 
```
grep "^>" genome_acheta_domesticus.fa | wc -l 
709385
```
The Genome length is around 929173017. So the GenomeLength/NumberOfReferences is equal 1309.829. I looked to this calcul following this issue advice <https://github.com/alexdobin/STAR/issues/103> 
So this calcul log2(929173017/709385) is equal to 10.35516

So following another recommandation of STAR, it is necessary to add those two parameter in ligne 312 of `runVARUS.pl`.
```--genomeChrBinNbits 10 --genomeSAindexNbases 13 ```

The command is then the following, don't forget to add the `Runlist.txt` inside the newly created folder `acheta_domesticus` since the createRunList is disable to put our custom `Runlist.txt` to align all the RNAseq data available for the five species.

```
mkdir varus ; cd varus 
# create or paste VARUSparameters.txt
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics --latinGenus=acheta --latinSpecies=domesticus --speciesGenome=../genome_acheta_domesticus.fa.masked --nocreateRunList
```

