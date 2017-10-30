## Install Ausgustus 


1. Download
```
wget http://bioinf.uni-greifswald.de/augustus/binaries/augustus-3.3.tar.gz
```
2. unpack

```
tar -xzvf augustus-3.3.tar.gz

The tar-archive contains one directory 'augustus-3.3' with the following sub-directories:
bin
src
include
config
examples
scripts
auxprogs
docs

```
3. compile

```
Install dependencies:
	
   - To turn on the optional support of gzip compressed input files you
    1) Edit common.mk and uncomment the line ZIPINPUT = true
    2) Install these dependencies:
       - Boost C++ Libraries: libboost-iostreams-dev (on Ubuntu: sudo apt-get install libboost-iostreams-dev)
       - zlib library for compression methods. Download from http://www.zlib.net/ or install via package manager
         (sudo apt-get install zlib1g-dev).
   
If comparative (multi-species, CGP) AUGUSTUS is enabled via uncommenting 
   COMPGENEPRED = true
   and 
   SQLITE = true or MYSQL = true
in common.mk then the following additional dependencies are required:

   - GNU scientific library (for eigen decompositions of matrices, sudo apt-get install libgsl-dev)
   - libsqlite3-dev or mysql++ (sudo apt-get install libmysql++-dev or sudo apt-get install libsqlite3-dev)
   - Boost C++ Library: sudo apt-get install libboost-graph-dev (at least version 1.45 from Nov. 2010)
   - integer linear program solver lpsolve: sudo apt-get install libsuitesparse-dev liblpsolve55-dev

Compiling bam2hints and filterBam requires:

   - The packages bamtools and libbamtools-dev
     (Ubuntu: sudo apt-get install bamtools libbamtools-dev,
      Possibly, you will need to add the repository http://us.archive.ubuntu.com/ubuntu vivid main universe
      to your /etc/apt/sources.list, first: 
      deb http://us.archive.ubuntu.com/ubuntu vivid main universe)

make

```
4. Set path 
```
export AUGUSTUS_CONFIG_PATH=~/bro_annotation/augustus/config/

export PATH=$PATH:~/bro_annotation/augustus/bin:~/bro_annotation/augustus/scripts/
```


