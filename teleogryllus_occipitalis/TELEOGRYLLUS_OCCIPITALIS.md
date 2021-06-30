## Teleogryllus occipitalis

**Retriving genome** 

I am using the assembly available on [NCBI](https://www.ncbi.nlm.nih.gov/genome/?term=txid470939[orgn]). It is also necessary to simplify the genome headers.

```
curl -L 'https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/011/170/035/GCA_011170035.1_Tocci_1.0/GCA_011170035.1_Tocci_1.0_genomic.fna.gz' --output gen_teleogryllus_occipitalis.fa.gz
gzip -d gen_teleogryllus_occipitalis.fa.gz 
sed 's/ /_/g' gen_teleogryllus_occipitalis.fa | sed 's/_Teleogryllus_occipitalis_DNA,//g' | sed 's/,_whole_genome_shotgun_sequence//g' | sed 's/_Teleogryllus_occipitalis_mitochondrial_DNA,_complete_genome//g' > genome_teleogryllus_occipitalis.fa && rm gen_teleogryllus_occipitalis.fa
```

**Busco on the assembly**
```
busco -m genome -i genome_teleogryllus_occipitalis.fa -o busco_public_assembly_occipitalis -l arthropoda_odb10 --cpu 12
```
```
        --------------------------------------------------
        |Results from dataset arthropoda_odb10            |
        --------------------------------------------------
        |C:97.3%[S:95.1%,D:2.2%],F:1.3%,M:1.4%,n:1013	  |  
	|985	Complete BUSCOs (C)			  |   
	|963	Complete and single-copy BUSCOs (S)	  |
	|22	Complete and duplicated BUSCOs (D)	  |
	|13	Fragmented BUSCOs (F)			  |
	|15	Missing BUSCOs (M)			  |
	|1013	Total BUSCO groups searched	          |
        --------------------------------------------------

```

**Run Repeatmodeler and RepeatMasker**

With the Dfam TE Tools Container activated :

```
mkdir database ; cd database 
BuildDatabase -name teleogryllus_occipitalis.DB -engine NCBI ../genome_teleogryllus_occipitalis.fa
RepeatModeler -database teleogryllus_occipitalis.DB -engine NCBI -pa 30

cd .. 
ln -s .
#change the folowing command with your path to the final consensi.fa.classified file 
/database/RM_23.MonJun71320472021/consensi.fa.classified 

RepeatMasker -pa 30 -lib consensi.fa.classified genome_teleogryllus_occipitalis.fa -xsmall
```

**Run VARUS**  
Paste the file `VARUSparameters.txt`, and don't forget to replace the `Runlist.txt` if you want to align all the RNAseq data available for the five species.
```
mkdir varus ; cd varus 
# create or paste VARUSparameters.txt
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics --latinGenus=teleogryllus --latinSpecies=occipitalis --speciesGenome=../genome_teleogryllus_occipitalis.fa.masked --nocreateRunList
# paste Runlist.txt inside the newly created gryllus_bimaculatus file
```

**Run BRAKER2** 

The command is run with arthropoda proteins and RNAseq librairies from the species studied here. 
```
cd .. ; mkdir braker2 ; cd braker2 ;
braker.pl --species=oceanicus --genome=../genome_teleogryllus_oceanicus.fa.masked --bam=../varus/teleogryllus_oceanicus/VARUS.bam --prot_seq=../../proteins/proteins.fasta --AUGUSTUS_ab_initio --softmasking --cores=14 --etpmode 
```

**Run BUSCO** 
For ab-initio with arthropoda : augustus.ab_initio.aa
```
busco -m proteins -i ./braker2/braker/augustus.ab_initio.aa -o busco_annotation_ab_i_occipitalis -l arthropoda_odb10 --cpu 16
```
```
        --------------------------------------------------
        |Results from dataset arthropoda_odb10            |
        --------------------------------------------------
        |C:7.3%[S:6.6%,D:0.7%],F:8.7%,M:84.0%,n:1013      |
        |74     Complete BUSCOs (C)                       |
        |67     Complete and single-copy BUSCOs (S)       |
        |7      Complete and duplicated BUSCOs (D)        |
        |88     Fragmented BUSCOs (F)                     |
        |851    Missing BUSCOs (M)                        |
        |1013   Total BUSCO groups searched               |
        --------------------------------------------------
```
```
busco -m proteins -i ./braker2/braker/augustus.hints.aa -o busco_annotation_occipitalis -l arthropoda_odb10 --cpu 16
```
Results in `short_summary.specific.arthropoda_odb10.busco_bimaculatus.txt`.
```
        --------------------------------------------------
        |Results from dataset arthropoda_odb10            |
        --------------------------------------------------
        |C:72.7%[S:66.5%,D:6.2%],F:11.9%,M:15.4%,n:1013   |
        |737    Complete BUSCOs (C)                       |
        |674    Complete and single-copy BUSCOs (S)       |
        |63     Complete and duplicated BUSCOs (D)        |
        |121    Fragmented BUSCOs (F)                     |
        |155    Missing BUSCOs (M)                        |        
		|1013   Total BUSCO groups searched               |
        --------------------------------------------------

```

**Investigating gtf annotations**

For Teleogryllus_occipitalis_geneset.gtf

```
wget 'https://s3-eu-west-1.amazonaws.com/pfigshare-u-files/20480874/Teleogryllus_occipitalis_geneset.gtf'
cut -f3 Teleogryllus_occipitalis_geneset.gtf | sort | uniq -c > output_T_occ.txt
```

```
... # many lignes 
 114824 CDS
 114861 exon
  94056 intron
  20293 start_codon
  20174 stop_codon
  20768 transcript
```
```
cut -f2 -d "m" output_T_occ.txt | sort | uniq | wc -l
20741
```

For braker.gtf 
```
cut -f3 ./braker2/braker/braker.gtf | sort | uniq -c
 670882 CDS
 670906 exon
 180789 gene
 486449 intron
 175999 start_codon
 179018 stop_codon
 182632 transcript
```

**To view the data on IGV** :
The genome : 
```
cat genome_teleogryllus_occipitalis.fa.masked  | sed -E 's/BLKR([0-9]+).1_//g' > genome_teleogryllus_occipitalis.fa.masked.strip
``` 
The output of braker : 
```
cat ./braker2/braker/braker.gtf | sed -E 's/BLKR([0-9]+).1_//g' > braker.gtf.strip
```

Charge `genome_teleogryllus_occipitalis.fa.masked.strip` as the genome, and `braker.gtf.strip` and `Teleogryllus_occipitalis_geneset.gtf` as tracks on IGV and you can explore results.  

![View1](docs/view1.png)  
View 1 : Whole view on scaffold 1  

![View2](docs/view2.png)  
View 2 : Zoomed view on scaffold 1 in the region between 1,748 and 1,871 kb  

![View3](docs/view3.png)  
View 3 : Zoomed view on scaffold 3 in the region between 1,667 and 1,841 kb
