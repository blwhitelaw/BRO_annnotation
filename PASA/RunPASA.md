# Running PASA

These instructions should be run on a machine with PASA installed

In order to do preliminary steps (Genome cleanup) bioawk is also needed. This can be installed with

```bash
sudo apt-get install git bison
git clone https://github.com/lh3/bioawk.git
cd bioawk
make
mv bioawk maketab ~/bin/
```

## Import data

Assuming we have a sister folder `assembly_data` we create symbolic links to raw assemblies as follows

```bash
ln -s ../assembly_data/hmac_dovetail_assembly.fasta .
ln -s ../assembly_data/hmac_dovetail_assembly.lengths.txt .
ln -s ../assembly_data/hapalochlaena_maculosa_na.fasta .
```

## Create a trimmed genome

According to genome assembly stats the vast majority of the genome is contained within a small number of long scaffolds. As a result we can remove scaffolds with lengths less than 20kb, which amounts to just over 2.5 percent of the total assembly size. By doing this we greatly streamline the annotation process. Indeed some steps will not work at all if the number of contigs is too large.

```bash
samtools faidx hmac_dovetail_assembly.fasta $(cat hmac_dovetail_assembly.lengths.txt | awk '$2>20000 {print $1}' | tr '\n' ' ') > hmac_dovetail_assembly_20k.fasta
```

## Build gmap indices

This is done using the gmap_build command.  In order to make it work we need to first split the genome into contigs

```bash
mkdir contigs
cd contigs
while read ctg;do samtools faidx ../hmac_dovetail_assembly_20k.fasta $ctg > ${ctg}.fasta;done < <(cat ../hmac_dovetail_assembly_20k.fasta | bioawk -c fastx '{print $name}')

cd ..
```

Now run the gmap_build command.  It is important that the name of the database match the name that PASA expects

```bash
nohup gmap_build -D . -d hmac_dovetail_assembly_20k.fasta.gmap contigs/*.fasta > gmap_build.log &
```


## Run PASA

```bash
${PASAHOME}/scripts/Launch_PASA_pipeline.pl -c pasa.annotationCompare.conf -C -R -g hmac_dovetail_assembly.fasta \
 -t hapalochlaena_maculosa_na.fasta --ALIGNERS gmap,blat --CPU 2
 ```

Extract ORFs for use as a training set in downstream analysis (Augustus)

```bash
${PASAHOME}/scripts/pasa_asmbls_to_training_set.dbi --pasa_transcripts_fasta bro_pasa.assemblies.fasta --pasa_transcripts_gff3 bro_pasa.pasa_assemblies.gff3
```

