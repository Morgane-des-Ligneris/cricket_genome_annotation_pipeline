# Pipeline for annotating cricket genome

This document relates the different steps realised to annotate cricket genome. 

# Contents

[INTRODUCTION](#introduction)

[PIPELINES](#pipelines) 
- [BRAKER2 INSTALLATION](#braker2-installation)

- [FUNNANOTATE INSTALLATION](##funnanotate-installation)

[OTHER SOFTWARES](#other-softwares-to-install)
    
# INTRODUCTION 

The first pipeline used is BRAKER, the other one will be Funannotate. 
For now only the common part of masking repat on the genome as been done to all species. 

The part are indicated either RUNNING or TO DO.

# PIPELINES

## BRAKER2 INSTALLATION
Install BRAKER2 following the instructions on the [gitHub](https://github.com/Gaius-Augustus/BRAKER).

### Tools 
BRAKER2 pipeline requieres several softwares including : 
- [AUGUSTUS](https://github.com/Gaius-Augustus/Augustus)
- [GeneMark-EX](http://exon.gatech.edu/GeneMark/license_download.cgi)
Install GeneMark-EX without forgeting the valid key in your home directory as the BRAKER [README](https://github.com/Gaius-Augustus/BRAKER#genemark-ex) recommand.
- BRAKER2's Perl modules dependencies can be installed with the following commands :
```
# for BRAKER 
sudo apt-get update -y 
sudo apt install perl
sudo apt-get install -y cpanminus 

cpanm File::Spec::Functions
cpanm Hash::Merge
cpanm List::Util
cpanm MCE::Mutex
cpanm Module::Load::Conditional
cpanm Parallel::ForkManager
cpanm YAML
cpanm Scalar::Util::Numeric
cpanm Math::Utils
cpanm threads

sudo apt-get install -y php-posix

# for GeneMark-EX
cpanm MCE::Mutex
cpanm Thread::Queue

# for Augustus 
sudo apt-get install libboost-iostreams-dev
sudo apt-get install zlib1g-dev
sudo apt-get install -y libgsl-dev
sudo apt-get install libboost-all-dev
sudo apt-get install libsuitesparse-dev
sudo apt-get install liblpsolve55-dev
sudo apt-get install libsqlite3-dev
sudo apt-get install libmysql++3v5
sudo apt-get install libbamtools-dev
sudo apt-get install libmysqlclient-dev
sudo apt-get install libmysql++-dev
```
- Python3
- [Bamtools](https://github.com/pezmaster31/bamtools)
- [NCBI BLAST+ or DIAMOND](https://github.com/Gaius-Augustus/BRAKER#ncbi-blast-or-diamond) 
- [ProHints](https://github.com/gatech-genemark/ProtHint)

### Optionnal tools installed 
- Samtools
- Biopython
- cdbfasta

## FUNNANOTATE INSTALLATION 

Funannotate is another pipeline that can be installed using conda. [Link for the installation](https://funannotate.readthedocs.io/en/latest/install.html)

# OTHER SOFTWARES TO INSTALL 
- [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic)
- [RepeatModeler and Repeat Masker](https://github.com/ISUgenomics/bioinformatics-workbook/blob/master/dataAnalysis/ComparativeGenomics/RepeatModeler_RepeatMasker.md). Installed using the [Dfam TE Tools Container](https://github.com/Dfam-consortium/TETools) with [docker](https://www.docker.com). Container activated with 
```
/home/ubuntu/data/mydatalocal/tools/dfam-tetools.sh --docker
```
- [STAR](http://manpages.ubuntu.com/manpages/focal/man1/STAR.1.html)
- [VARUS](https://github.com/Gaius-Augustus/VARUS)
- [SRA-toolkit](https://github.com/ncbi/sra-tools/wiki/02.-Installing-SRA-Toolkit)

# GENOME ANNOTATION USING BRAKER2
The instructions and steps bellow follow the ["Tutorial on how to run run BRAKER2 gene prediction pipeline"](https://bioinformaticsworkbook.org/dataAnalysis/GenomeAnnotation/Intro_to_Braker2.html#gsc.tab=0)
STEPS : 
- Creating repeats library and sofstmasking the genome with Repeat Masker. 
- Trimming and aligning RNAseq data. 
...

**Protein database preparation**

Because there is not enough RNA-seq libraries available. It is possible to try the approach of BRAKER2 with proteins of any evolutionary distance. On Braker github : "We recommend using OrthoDB as basis for proteins.fa. The instructions on how to prepare the input OrthoDB proteins are documented here: https://github.com/gatech-genemark/ProtHint#protein-database-preparation."

As the genome of interest is an insect, we can try with arthropoda proteins. This is inside the `crickets`folder.
```
mkdir proteins ; cd proteins ; 
wget https://v100.orthodb.org/download/odb10_arthropoda_fasta.tar.gz
tar xvf odb10_arthropoda_fasta.tar.gz
cat arthropoda/Rawdata/* > proteins.fasta
```

**Retriving RNAseq libraries using VARUS**

Inside the each species folder, from the description on how to run VARUS it is first necessary to copy the file `VARUSparameters.txt` from the example folder to the working directory. Or you can create the file. 
```
mkdir varus ; cd varus 
touch VARUSparameters.txt 
```
And paste the following inside it: 
```
--batchSize 50000
--blockSize 5000
--components 1
--cost 0.001
--deleteLater 0
--estimator 2
--exportObservationsToFile 1
--exportParametersToFile 1
--fastqDumpCall fastq-dump
--genomeDir ./genome/
--lambda 10.0
--lessInfo 1
--loadAllOnce 0
--maxBatches 300
--mergeThreshold 10
--outFileNamePrefix ./
--pathToParameters ./VARUSparameters.txt
--pathToRuns ./
--pathToVARUS /home/ubuntu/data/mydatalocal/tools/VARUS/Implementation/
--profitCondition 0
--pseudoCount 1
--qualityThreshold 1
--randomSeed 1
--readParametersFromFile 1
--runThreadN 10
--verbosityDebug 1

```
The `--qualityThreshold` was diminished to 1 because it is not only the RNAseq data from the studied specie but also the other one chosen here. 

After you can run the command indicated in each specie sections. Following that in order to align RNAseq data from all the species and not just itself, you have to go inside the newly created folder with the name of the specie and replace the `Runlist.txt` with the following content. It is all the accession from the five species selected.
This needs to be done during the indexing of the genome, otherwise it will be too late once the program starts to map the data.
```
@Run_acc	total_spots	total_bases	avg_len	bool:paired	color_space
DRR201341	21463942	4335716284	202	1	0	
SRR7692599	4493126	1535145791	341.66	1	0	
SRR7692600	7361666	2645830546	359.4	1	0	
SRR7692601	12026418	4339443659	360.82	1	0	
SRR7692602	11518478	4155086317	360.73	1	0	
SRR7692603	11277632	4159288002	368.8	1	0	
SRR7692604	10142769	3710996250	365.87	1	0	
SRR7692605	11140847	4043827924	362.97	1	0	
SRR6250553	16577170	4973151000	300	1	0	
SRR6250554	13433800	4030140000	300	1	0	
SRR6001368	211993742	63598122600	300	0	0	
SRR2230498	8449300	2534790000	300	1	0	
SRR1552491	41452859	4062380182	98	1	0	
SRR14026720	16203900	4867811794	300.4	1	0	
SRR14026721	17732526	5328329715	300.48	1	0	
SRR14026722	15666493	4708117904	300.52	1	0	
SRR14026723	15400493	4627529968	300.47	1	0	
SRR14026724	18664030	5606827444	300.4	1	0	
SRR14026725	12816612	3851356902	300.49	1	0	
SRR14026726	17461709	5244412457	300.33	1	0	
DRR258410	631519	115233688	182.47	1	0	
DRR258409	804528	149743542	186.12	1	0	
DRR258408	854191	171361584	200.61	1	0	
DRR258407	1085690	199485064	183.74	1	0	
DRR258406	739031	132792466	179.68	1	0	
DRR258405	769263	149859888	194.8	1	0	
DRR258404	775810	150076603	193.44	1	0	
DRR258403	1098177	199391064	181.56	1	0	
DRR258402	4829903	911398961	188.69	1	0	
DRR258401	1019346	204657665	200.77	1	0	
DRR258400	1257820	234918985	186.76	1	0	
DRR258399	836620	164193333	196.25	1	0	
DRR258398	1483936	286974341	193.38	1	0	
DRR258397	797529	157167449	197.06	1	0	
DRR258396	922971	177537002	192.35	1	0	
DRR258395	783806	155222828	198.03	1	0	
DRR258394	534100	106071982	198.59	1	0	
DRR258393	391055	77092821	197.14	1	0	
DRR258392	541870	103036682	190.15	1	0	
DRR258391	1620799	321928029	198.62	1	0	
DRR258390	252579	47302277	187.27	1	0	
DRR258389	722017	143037406	198.1	1	0	
DRR258388	1943323	356821113	183.61	1	0	
DRR258387	957383	191041152	199.54	1	0	
DRR255768	154321356	33038672200	214.09	1	0	
DRR255767	83111061	16622212200	200	1	0	
DRR255766	38110022	7622004400	200	1	0	
DRR255765	123373139	26328173000	213.4	1	0	
DRR255764	24888187	6222046750	250	1	0	
DRR255763	79678193	16355562900	205.27	1	0	
SRR12326205	36883235	11064970500	300	1	0	
SRR12326206	34141547	10242464100	300	1	0	
SRR12326207	31236520	9370956000	300	1	0	
SRR12326208	35469114	10640734200	300	1	0	
SRR12326209	29075720	8722716000	300	1	0	
SRR12326210	42130046	12639013800	300	1	0	
SRR12326211	46663306	13998991800	300	1	0	
SRR12326212	28903930	8671179000	300	1	0	
SRR12326213	33449104	10034731200	300	1	0	
SRR12326214	30397636	9119290800	300	1	0	
SRR12326215	27584303	8275290900	300	1	0	
SRR12326216	22348689	6704606700	300	1	0	
SRR12177798	45640572	13746978434	301.2	1	0	
SRR12177799	41591645	12525999722	301.16	1	0	
SRR12177800	43902098	13223943579	301.21	1	0	
SRR12177801	70116591	21116689356	301.16	1	0	
SRR12177802	47814783	14401891672	301.2	1	0	
SRR12177803	41171831	12399142024	301.15	1	0	
SRR12177804	16462746	4957993612	301.16	1	0	
SRR12177805	50371238	15170314160	301.17	1	0	
SRR12177806	39345868	11850661451	301.19	1	0	
SRR12177807	37890238	11412322302	301.19	1	0	
SRR12177808	19313571	5816954346	301.18	1	0	
SRR12177809	49590954	14936896543	301.2	1	0	
SRR12177810	43339537	13053282552	301.18	1	0	
SRR12177811	33937820	10220821444	301.16	1	0	
SRR12177812	40031615	12056479786	301.17	1	0	
SRR12177813	34061899	10259907035	301.21	1	0	
SRR12177814	53149576	16007414554	301.17	1	0	
SRR12177815	35466852	10681764603	301.17	1	0	
SRR12177816	17355762	5226970944	301.16	1	0	
SRR12177817	21294047	6413180608	301.17	1	0	
SRR12177818	43893107	13219957438	301.18	1	0	
SRR12177819	24596912	7407182279	301.14	1	0	
SRR12177820	47703870	14369367359	301.22	1	0	
SRR12177821	27337778	8232731930	301.14	1	0	
SRR12177822	31923277	9613811782	301.15	1	0	
SRR12177823	42313736	12745304479	301.2	1	0	
SRR12177824	32836812	9889660212	301.17	1	0	
SRR12177825	39929741	12026954182	301.2	1	0	
SRR12177826	47306966	14247386521	301.16	1	0	
SRR12177827	51131929	15401397153	301.2	1	0	
SRR7892536	42694930	8538986000	200	1	0	
SRR7892537	57107448	11421489600	200	1	0	
SRR7892538	71525642	10799043857	150.98	1	0	
SRR5358911	58876041	11775208200	200	1	0	
SRR5358910	64927442	12985488400	200	1	0	
SRR5358909	61500613	12300122600	200	1	0	
SRR5358908	68395006	13679001200	200	1	0	
SRR5358907	59670374	11934074800	200	1	0	
SRR5358906	51341726	10268345200	200	1	0	
SRR5358905	45136951	9027390200	200	1	0	
SRR5358904	60668422	12133684400	200	1	0	
SRR5358903	60388499	12077699800	200	1	0	
SRR5358902	63149691	12629938200	200	1	0	
SRR5358901	64491826	12898365200	200	1	0	
SRR5358900	68066557	13613311400	200	1	0	
SRR5358899	43648742	8729748400	200	1	0	
SRR5358898	33263487	6652697400	200	1	0	
SRR5358897	68726540	13745308000	200	1	0	
SRR5358896	61475571	12295114200	200	1	0	
SRR5358895	60450357	12090071400	200	1	0	
SRR5358894	64617453	12923490600	200	1	0	
SRR5358893	61922328	12384465600	200	1	0	
SRR5358892	71483111	14296622200	200	1	0	
SRR5358891	74859468	14971893600	200	1	0	
DRR001986	11841814	592090700	50	0	0	
DRR001985	10845142	542257100	50	0	0	
SRR060816	4102061	2359984750	575.31	0	0	
SRR060815	67352	15481108	229.85	0	0	
SRR060814	78935	18370610	232.73	0	0
ERR4552772	22075201	6404370618	290.11	1	0	
ERR4552771	47187286	13791265661	292.26	1	0	
ERR4552770	52381020	15090688921	288.09	1	0	
ERR4552769	44998719	12978664618	288.42	1	0	
ERR4552768	43152470	12469054258	288.95	1	0	
ERR4552767	55998773	16205909414	289.39	1	0	
ERR4552766	43785761	12647310209	288.84	1	0	
ERR4552765	43425421	12646190223	291.21	1	0	
ERR4552764	40409944	11755431927	290.9	1	0	
ERR4552763	60486111	17486090984	289.09	1	0	
ERR4552762	58663190	17013521421	290.02	1	0	
ERR4552761	49486313	14345904470	289.89	1	0	
ERR4552760	40007211	11485973229	287.09	1	0	
ERR4552759	44134837	12653678299	286.7	1	0	
ERR4552758	41930507	11822257728	281.94	1	0	
ERR4552757	31397679	8876636923	282.71	1	0	
ERR4552756	37937522	10741659105	283.14	1	0	
ERR4552755	41019202	11617524932	283.22	1	0	
ERR4552754	35872516	10054427242	280.28	1	0	
ERR4552753	39017900	10780868502	276.3	1	0	
ERR4552752	45815483	13183120938	287.74	1	0	
ERR4552751	37467840	10552755044	281.64	1	0	
ERR4552750	36219328	10304763247	284.51	1	0	
ERR4552749	25110215	7008109804	279.09	1	0	
SRR11889021	10143365	1034623230	102	1	0	
SRR11889020	10374333	1058181966	102	1	0	
SRR11889019	8884628	906232056	102	1	0	
SRR11889018	10980372	1119997944	102	1	0	
SRR11889017	10220700	1042511400	102	1	0	
SRR11889016	9276082	946160364	102	1	0	
SRR11889015	11579676	1181126952	102	1	0	
SRR11889014	10102515	1030456530	102	1	0	
SRR11889013	10583669	1079534238	102	1	0	
SRR11889012	10842403	1105925106	102	1	0	
SRR11889011	11422717	1165117134	102	1	0	
SRR11889010	11462502	1169175204	102	1	0	
SRR11889009	9918416	1011678432	102	1	0	
SRR11889008	10488789	1069856478	102	1	0	
SRR11889007	11074436	1129592472	102	1	0	
SRR11889006	9605375	979748250	102	1	0	
SRR11889005	11006072	1122619344	102	1	0	
SRR11889004	10762560	1097781120	102	1	0	
ERR2619847	28721838	5378123343	187.24	1	0	
ERR2619846	24456228	4633818298	189.47	1	0	
ERR2619845	35552881	6583772962	185.18	1	0	
ERR2619844	35382796	6621294510	187.13	1	0	
ERR2619843	22295013	4187114180	187.8	1	0	
ERR2619842	23843272	4440649294	186.24	1	0	
ERR2619841	29544754	5505372877	186.34	1	0	
ERR2619840	17229408	3293481726	191.15	1	0	
ERR2619839	9172067	1755270177	191.37	1	0	
ERR2619838	33665646	6319482151	187.71	1	0	
ERR2619837	24262153	4519820211	186.29	1	0	
ERR2619836	30002776	5697941685	189.91	1	0	
ERR2619835	20029845	3742255454	186.83	1	0	
ERR2619834	30140953	5616898551	186.35	1	0	
ERR2619833	18133920	3481935711	192.01	1	0	
ERR2619832	27009996	5118334320	189.49	1	0	
ERR2619831	20720037	3972261536	191.71	1	0	
ERR2619830	17431719	3314529158	190.14	1	0	
ERR2619829	20851634	3948574243	189.36	1	0	
ERR2619828	17526777	3347807520	191.01	1	0	
ERR2619827	15689092	2994368672	190.85	1	0	
ERR2619826	17324645	3323687015	191.84	1	0	
ERR2619825	28215093	5355020590	189.79	1	0	
ERR2619824	20167484	3839467302	190.37	1	0	
ERR2619823	18981007	3547342622	186.88	1	0	
ERR2619822	18827916	3553128048	188.71	1	0	
ERR2619821	16913615	3156045332	186.59	1	0	
ERR2619820	20390600	3869607238	189.77	1	0	
ERR2619819	22753029	4114955310	180.85	1	0	
ERR2619818	20723620	3920457823	189.17	1	0	
ERR2619817	22255296	4211433605	189.23	1	0	
ERR2619816	22526496	4268565975	189.49	1	0	
ERR2619815	19152513	3701039100	193.24	1	0	
ERR2619814	22253685	4237484370	190.41	1	0	
ERR2619813	16016686	3041154051	189.87	1	0	
ERR2619812	25750536	4919639389	191.04	1	0	
ERR2619811	30135313	5633500278	186.94	1	0	
ERR2624516	19637927	3716680606	189.26	1	0	
ERR2624515	15185523	2893353952	190.53	1	0	
ERR2624514	17278152	3298726219	190.91	1	0	
ERR2624513	21009566	3906534446	185.94	1	0	
ERR2624512	16211305	3096038148	190.98	1	0	
ERR2624511	22421385	4174748112	186.19	1	0	
ERR2624510	19267495	3586169369	186.12	1	0	
ERR2624509	22662186	4160420876	183.58	1	0	
ERR2624508	21391717	4004436178	187.19	1	0	
ERR2624507	30119603	5651456932	187.63	1	0	
ERR2624506	22673794	4233506745	186.71	1	0	
ERR2624505	28027066	5039557661	179.81	1	0	
SRR4292230	19178113	3551149407	185.16	1	0	
SRR4292229	19426286	3533490322	181.89	1	0	
SRR4292228	31101990	5721666779	183.96	1	0	
SRR4292227	26621039	4932207606	185.27	1	0	
SRR4292226	21409818	3999304425	186.79	1	0	
SRR4292225	19876536	3706609079	186.48	1	0	
SRR4292224	21519773	4109427269	190.96	1	0	
SRR4292223	20314203	3877672004	190.88	1	0	
SRR4292222	19729851	3758686090	190.5	1	0	
SRR4292221	16046684	3016305815	187.97	1	0	
SRR4292220	18909722	3544210334	187.42	1	0	
SRR4292219	20970276	3978597503	189.72	1	0	
SRR4292218	20252790	3849829420	190.08	1	0	
SRR4292217	24869694	4570011442	183.75	1	0	
SRR4292216	20748837	3929580076	189.38	1	0	
SRR4292215	19581222	3698026013	188.85	1	0	
SRR4292214	21289617	4053816643	190.41	1	0	
SRR4292213	20234480	3813695107	188.47	1	0	
SRR4292212	16909101	3211504449	189.92	1	0	
SRR4292211	13930418	2654889953	190.58	1	0	
SRR4292210	18167764	3421655186	188.33	1	0	
SRR4292209	18675819	3553352137	190.26	1	0	
SRR4292208	22008624	4185226498	190.16	1	0	
SRR4292207	24735866	4686186989	189.44	1	0	
SRR4292205	30521397	5687557580	186.34	1	0	
SRR4292204	20735523	3957568239	190.85	1	0	
SRR4292203	25297584	4757045258	188.04	1	0	
SRR4292202	25508876	4790439591	187.79	1	0	
SRR4292201	22165232	4148700210	187.17	1	0	
SRR4292200	26530364	5020819747	189.24	1	0	
SRR4292199	23711773	4470120339	188.51	1	0	
SRR4292198	23497415	4528034778	192.7	1	0	
SRR4292197	24870404	4767185592	191.68	1	0	
SRR4292196	21375323	4090550454	191.36	1	0	
SRR4292195	19618474	3770354230	192.18	1	0	
SRR4292194	25947709	4833253346	186.26	1	0	
SRR4292193	23015914	4428857588	192.42	1	0	
SRR4292192	23852141	4620224292	193.7	1	0	
SRR4292191	17471064	3333936228	190.82	1	0	
SRR4292190	17429104	3313802628	190.13	1	0	
SRR4292189	21415093	4116820185	192.23	1	0	
SRR4292188	19030316	3672178774	192.96	1	0	
SRR4292187	16392856	3151204517	192.23	1	0	
SRR4292186	21409018	4110892367	192.01	1	0	
SRR4292185	25671123	4754439881	185.2	1	0	
SRR4292184	19878742	3688156407	185.53	1	0	
SRR4292183	22385500	4133340003	184.64	1	0	
SRR4292182	24948575	4694435112	188.16	1	0	
SRR2034761	22089619	4204947227	190.35	1	0	
SRR2034751	25601393	4880632382	190.63	1	0	
SRR2034750	20827273	3954754214	189.88	1	0	
SRR2034760	14589820	2773303014	190.08	1	0	
SRR2034759	18371918	3506651514	190.87	1	0	
SRR2034758	20452626	3864911009	188.96	1	0	
SRR2034757	24173564	4618421343	191.05	1	0	
SRR2034756	21541433	4103478415	190.49	1	0	
SRR2034755	24485287	4680830964	191.16	1	0	
SRR2034754	22724078	4380753330	192.78	1	0	
SRR2034753	23217896	4435199187	191.02	1	0	
SRR2034752	33357915	6221730361	186.51	1	0	
SRR329505	293976	129113845	439.19	0	0	
SRR329504	295416	132494848	448.5	0	0	
SRR329503	357410	159603045	446.55	0	0	
SRR329502	288814	124734094	431.88	0	0

```

## Acheta domesticus

**Retriving genome** 

I am using the assembly available on the [NCBI](https://www.ncbi.nlm.nih.gov/genome/?term=txid6997[orgn]). It is necessary to simplify the genome headers because it can create errors later in the pipeline. There need to be no spaces or odd caracters, the size need to be less than 50 characters. 
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

## Gryllus bimaculatus

**Retriving genome** 

I am using the assembly available on this [platform](http://gbimaculatusgenome.rc.fas.harvard.edu). 
```
curl https://gbimaculatusgenome.rc.fas.harvard.edu/session/c044a9e5a74f128dbd50738d0ebe0dbc/download/downloadDataGenome?w= --output genome_gryllus_bimaculatus.fa
```
**Run Repeatmodeler and RepeatMasker**

```
mkdir database ; cd database 
BuildDatabase -name gryllus_bimaculatus.DB -engine NCBI ../genome_gryllus_bimaculatus.fa
RepeatModeler -database gryllus_bimaculatus.DB -engine NCBI -pa 20

cd .. 
ln -s ./database/RM_23.FriMay281932292021/consensi.fa.classified
RepeatMasker -pa 20 -gff -lib consensi.fa.classified genome_gryllus_bimaculatus.fa
```

**Run VARUS**

The command is the following, for this one maybe this is not necessary to replace  the `Runlist.txt` because there is 108 RNAseq librairies available already. 
```
mkdir varus ; cd varus
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics \
  --latinGenus=gryllus --latinSpecies=bimaculatus \
  --speciesGenome=../genome_gryllus_bimaculatus.fa 
```

**Run BRAKER2** RUNNING

Running with arthropoda proteins and bam file alligned against all RNAseq librairies available for this species. RNAseq data from other species was not used because there was already enough librairies for this specie (108 available on the NCBI). (The file VARUS.bam here was copied from another simple run of varus on the specie nampe.)
```
cd .. ; mkdir braker2 ; cd braker2 ;
perl /home/ubuntu/data/mydatalocal/tools/BRAKER/scripts/braker.pl --genome=../genome_gryllus_bimaculatus.fa.masked --bam=../varus/VARUS.bam --prot_seq=../proteins/proteins.fasta --softmasking --cores=8 --etpmode 
```

## Laupala kohenlensis

**Retriving genome** 

I am using the assembly available on the [NCBI](https://www.ncbi.nlm.nih.gov/genome/?term=txid109027[orgn]). 
```
curl https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/002/313/205/GCA_002313205.1_ASM231320v1/GCA_002313205.1_ASM231320v1_genomic.fna.gz --output genome_laupala_kohalensis.fa.gz
gzip -d genome_laupala_kohalensis.fa.gz 
sed 's/ /_/g' gen_laupala_kohalensis.fa | sed 's/,_whole_genome_shotgun_sequence//g' | sed 's/_Laupala_kohalensis_isolate_Lakoh051//g' > genome_laupala_kohalensis.fa && rm gen_laupala_kohalensis.fa
```
**Run Repeatmodeler and RepeatMasker** TO DO 

```
mkdir database ; cd database
BuildDatabase -name DB_laupala_kohalensis -engine NCBI ../genome_laupala_kohalensis.fa
RepeatModeler -database DB_laupala_kohalensis -engine NCBI -pa 20

cd .. 
ln -s ../database/RM_47112.SunMay231510042021/consensi.fa.classified
RepeatMasker -pa 20 -gff -lib consensi.fa.classified genome_teleogryllus_occipitalis.fa
```

**Run VARUS** TO DO 

The command is the following, don't forget to replace the `Runlist.txt` if you want to align all the RNAseq data available for the five species. 
For this specie we are 'tcheating', because there is no RNAseq data available but I still want to see if RNAseq data from other species is aligning I am putting intentionnaly the wrong name since we are anyway changing the `Runlist.txt` file. 
```
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics \
  --latinGenus=teleogryllus --latinSpecies=occipitalis \
  --speciesGenome=../genome_laupala_kohalensis.fa  
```

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

**Run VARUS** RUNNING 

The command is the following, don't forget to replace the `Runlist.txt` if you want to align all the RNAseq data available for the five species.
```
/home/ubuntu/data/mydatalocal/tools/VARUS/runVARUS.pl --aligner=STAR --readFromTable=0 --createindex=1 --runThreadN 8 --createStatistics \
  --latinGenus=teleogryllus --latinSpecies=occipitalis \
  --speciesGenome=../genome_teleogryllus_occipitalis.fa 
```

**Run BRAKER2**

First test with arthropoda proteins and RNAseq librairies from the species studied here. 
```
cd .. ; mkdir braker2 ; cd braker2 ;
perl /home/ubuntu/data/mydatalocal/tools/BRAKER/scripts/braker.pl --genome=../genome_teleogryllus_occipitalis.fa.masked --bam=../varus2/teleogryllus_occipitalis/VARUS.bam --prot_seq=../../proteins/proteins.fasta --softmasking --cores=8 --etpmode 
```

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


