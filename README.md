# Annotation of the Hapalochlanea maculosa genome

This repository contains all scripts and code required to reproduce the annotation of the Blue Ringed Octopus genome. 

# Downloading Data

See the [README](assembly_data) within the assembly_data directory for details on how to download the genome assembly and transcript data for annotation

# Summary Statistics

These are provided as an RStudio project `BRO_annotation.Rproj` and associated R Markdown files

- [contig_lengths](contig_lengths.Rmd) summarises lengths of scaffolds and picks a cutoff for filtering small scaffolds from the assembly

# Installation and Setup

The easiest option for setup is to use the provided Dockerfile. This will create a docker image on your system with all dependencies installed. 

Build the docker image if necessary. If the image has already been built on your system you will see it appear if you run `docker images`. It should be called `broannotation`.

```bash
cd PASA
docker build -t iracooke:broannotation .
```

Start a mysql image to run the database. The `-v` option will use a specific docker volume to store the data. If you restart the mysql image with this same volume it will have access to the same data.

```bash
sudo docker run --name pasamysql --env-file mysql.env -v pasamysql-vol:/var/lib/mysql -d mysql:5
```

After the mysql image is running you can start the broannotation image. Run the image within the folder that contains all of the necessary files, in this case the folder is called PASA. The `--link` option will allow the broannotation image to access the mysql image regardless of the IP address that the mysql image gets. The `- v ` option will link a directory called PASA where files can be made accessible to the docker container. 

```bash
docker run -d -it --link pasamysql:mysql -v $(pwd):/PASA iracooke:broannotation
```

In order to access the container run the code below with the appropriate name. The name can be accessed by running `sudo docker ps`.

```bash
sudo docker exec -it "container name" bash
```

Test that it worked by running the following commands from within the broannotation image.

```bash
cd $PASAHOME/sample_data

../scripts/Launch_PASA_pipeline.pl -c alignAssembly.config -C -R -g genome_sample.fasta -t all_transcripts.fasta.clean -T -u all_transcripts.fasta -f FL_accs.txt --ALIGNERS blat,gmap --CPU 2
```

Alternatively you prefer to install everything on a bare-metal machine follow these [instructions to install PASA](PASA/InstallPASA.md)


# Running the Annotation Pipeline

1. [Run the PASA pipeline](PASA/RunPASA.md) from within the PASA docker image (see above)

