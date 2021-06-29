## Teleogryllus occipitalis

**Retriving genome** 

I am using the assembly available on [NCBI](https://www.ncbi.nlm.nih.gov/genome/?term=txid470939[orgn]). 

```
curl -L 'https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/011/170/035/GCA_011170035.1_Tocci_1.0/GCA_011170035.1_Tocci_1.0_genomic.fna.gz' --output gen_teleogryllus_occipitalis.fa.gz
gzip -d gen_teleogryllus_occipitalis.fa.gz 
sed 's/ /_/g' gen_teleogryllus_occipitalis.fa | sed 's/_Teleogryllus_occipitalis_DNA,//g' | sed 's/,_whole_genome_shotgun_sequence//g' | sed 's/_Teleogryllus_occipitalis_mitochondrial_DNA,_complete_genome//g' > genome_teleogryllus_occipitalis.fa && rm gen_teleogryllus_occipitalis.fa
```

**Busco on the assembly**
```
docker run -u $(id -u) -v $(pwd):/busco_wd ezlabgva/busco:v5.1.3_cv1 busco -m genome -i genome_teleogryllus_occipitalis.fa -o busco_public_assembly_occipitalis -l arthropoda_odb10 --cpu 12
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
ln -s ./database/RM_23.MonJun71320472021/consensi.fa.classified # use it with your consenti.fa.classified path
RepeatMasker -pa 30 -lib consensi.fa.classified genome_teleogryllus_occipitalis.fa -xsmall
```

**Run VARUS**  
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
perl /home/ubuntu/data/mydatalocal/tools/BRAKER/scripts/braker.pl --species=oceanicus --genome=../genome_teleogryllus_oceanicus.fa.masked --bam=../varus/teleogryllus_oceanicus/VARUS.bam --prot_seq=../../proteins/proteins.fasta --AUGUSTUS_ab_initio --softmasking --cores=1 --etpmode 
```

**Run BUSCO** 

```
docker run -u $(id -u) -v $(pwd):/busco_wd ezlabgva/busco:v5.1.3_cv1 busco -m proteins -i ./braker2/braker/augustus.hints.aa -o busco_annotation_occipitalis -l arthropoda_odb10 --cpu 16
```

RESULTS in `short_summary.specific.arthropoda_odb10.busco_bimaculatus.txt`.

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
For Ab-initio with arthropoda : augustus.ab_initio.aa
```
docker run -u $(id -u) -v $(pwd):/busco_wd ezlabgva/busco:v5.1.3_cv1 busco -m proteins -i ./braker2/braker/augustus.ab_initio.aa -o busco_annotation_ab_i_occipitalis -l arthropoda_odb10 --cpu 16
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
docker run -u $(id -u) -v $(pwd):/busco_wd ezlabgva/busco:v5.1.3_cv1 busco -m proteins -i ./braker2/braker/augustus.hints.aa -o busco_annotation_occipitalis_insecta -l insecta_odb10 --cpu 16
```
```
        --------------------------------------------------
        |Results from dataset insecta_odb10               |
        --------------------------------------------------
        |C:73.3%[S:66.4%,D:6.9%],F:10.7%,M:16.0%,n:1367   |
        |1003   Complete BUSCOs (C)                       |
        |908    Complete and single-copy BUSCOs (S)       |
        |95     Complete and duplicated BUSCOs (D)        |
        |146    Fragmented BUSCOs (F)                     |
        |218    Missing BUSCOs (M)                        |
        |1367   Total BUSCO groups searched               |
        --------------------------------------------------
```

**Investigating gtf annotations**

For Teleogryllus_occipitalis_geneset.gtf

```
cut -f3 Teleogryllus_occipitalis_geneset.gtf | sort | uniq -c > output_T_occ.txt
```

```
1 # This transcript jg10.t1 is derived from g48810.t1 from the input file /work/kataoka/BRAKER/191206_1/augustus.E.gtf
...
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

cat genome_teleogryllus_occipitalis.fa.masked  | sed -E 's/BLKR([0-9]+).1_//g' > genome_teleogryllus_occipitalis.fa.masked.strip