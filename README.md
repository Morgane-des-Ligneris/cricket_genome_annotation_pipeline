# Pipeline for annotating cricket genome

This document relates each step of this pipeline to annotate cricket genomes. 

# Contents

[INTRODUCTION](#introduction)  
[PIPELINES](#pipelines) 
    
# INTRODUCTION 

The first pipeline used is BRAKER, the other one will be  

# PIPELINES

## BRAKER 
Install BRAKER following the instructions on the [gitHub link](https://github.com/Gaius-Augustus/BRAKER) and follow all the instructions. 

### Mandatory tools 
BRAKER pipeline requiere several softwares including : 
- [AUGUSTUS](https://github.com/Gaius-Augustus/Augustus)
- [GeneMark-EX](http://exon.gatech.edu/GeneMark/license_download.cgi)
Install GeneMark-EX without forgeting the valid key in your home directory as the BRAKER [README](https://github.com/Gaius-Augustus/BRAKER#genemark-ex) recommand.

```bash 
wget http://topaz.gatech.edu/GeneMark/tmp/GMtool__ZHQu/gmes_linux_64.tar.gz 
tar xzvf gmes_linux_64.tar.gz
cd 
wget http://topaz.gatech.edu/GeneMark/tmp/GMtool__ZHQu/gm_key_64.gz 
gunzip gm_key_64.gz
mv gm_key_64 .gm_key
```

- All Cpan-Perl modules that are required :

```
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

sudo apt install samtools libhts-dev

```

- Biopython
```
sudo apt-get install python3-pip
sudo pip3 install biopython
```

-  cdbfasta
```
sudo apt-get install cdbfasta
```

### Path update on .bashrc following the indications 
```
PATH=/home/ubuntu/data/mydatalocal/tools/BRAKER/scripts/:$PATH
export PATH

export /home/ubuntu/data/mydatalocal/tools/gmes_linux_64/

export PATH=/home/ubuntu/data/mydatalocal/tools/Augustus/bin:/home/ubuntu/data/mydatalocal/tools/Augustus/scripts:$PATH
export AUGUSTUS_CONFIG_PATH=/home/ubuntu/data/mydatalocal/tools/Augustus/config/

export PYTHON3_PATH=/usr/bin/python3/

export PROTHINT_PATH=/home/ubuntu/data/mydatalocal/tools/ProHint/

export SAMTOOLS_PATH=/usr/bin/samtools

export MAKEHUB_PATH=/home/ubuntu/data/mydatalocal/tools/MakeHub
```

## Funannotate

Funannotate is another pipeline that can be installed using conda. 




